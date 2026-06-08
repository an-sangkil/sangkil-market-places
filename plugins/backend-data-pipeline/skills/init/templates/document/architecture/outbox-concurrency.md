# Outbox 동시성 — Cloud Run 환경 제약과 대응

> mydata-collector의 Outbox 패턴이 GCP Cloud Run 위에서 동작할 때 발생하는 중복 처리/Race Condition 이슈와 대응 방안을 정리한다.

## 1. 배경

### 현재 Outbox 구조

- 테이블: `mydata.outbox_events`
  - 상태 컬럼: `status` (`NONE → READY → SENT` / `FAILED`)
  - 락/소유권 컬럼 없음 (worker_id, version, locked_at 등 부재)
- 코드 위치:
  - `OutboxInitializer.java` — 애플리케이션 시작 시 데몬 스레드 1개로 `OutboxWorker.run()` 호출
  - `OutboxWorker.java` — 폴링 루프 (`synchronized(lock)`로 JVM 내부 단일 스레드 처리)
  - `OutboxEventJpaRepository.nextBatchReady` — `SELECT ... WHERE status = :status AND published_time <= :now ORDER BY id ASC LIMIT N`
    - **`FOR UPDATE` / `SKIP LOCKED` 없음**
- 처리 흐름:
  1. `status=NONE` row 최대 10개 SELECT
  2. 각 이벤트마다 `NONE → READY` UPDATE
  3. Pub/Sub publish
  4. `READY → SENT` UPDATE
  5. 예외 발생 시 `FAILED`

### 배포 환경

- GCP Cloud Run (HTTP/이벤트 트래픽 기반 오토스케일)
- `min_instances`, `max_instances` 설정에 따라 인스턴스 수가 1~N으로 동적 변동
- 인스턴스 간 공유 메모리/락 없음. 공유 상태는 MySQL뿐.

## 2. 동시성 이슈

`OutboxWorker`의 `synchronized(lock)`은 **같은 JVM(같은 Cloud Run 인스턴스) 내부에서만** 동작한다.
인스턴스가 2개 이상이면 다음 시나리오가 모두 가능하다.

### 2.1 중복 publish (가장 심각)

```
T0  인스턴스 A: SELECT ... WHERE status='NONE' → id=[1,2,3]
T0' 인스턴스 B: SELECT ... WHERE status='NONE' → id=[1,2,3]  (동일 결과)
T1  A: id=1 status='READY' UPDATE
T1' B: id=1 status='READY' UPDATE  (덮어쓰기, 두 번째도 성공)
T2  A: Pub/Sub publish (id=1)
T2' B: Pub/Sub publish (id=1)      ← 동일 이벤트 중복 발행
T3  A: id=1 status='SENT'
T3' B: id=1 status='SENT'
```

- `nextBatchReady` 쿼리에 row-level lock이 없으므로, 두 인스턴스가 같은 row 집합을 동시에 읽는다.
- `status='READY'` UPDATE는 조건 없이 무조건 덮어쓰므로 첫 번째/두 번째 UPDATE가 모두 통과한다.
- 결과: **동일 outbox 이벤트가 두 번 이상 Pub/Sub로 발행됨** → 다운스트림 (`JobInitializer` 등)이 중복 수신.

### 2.2 status 전이 Race

```
T0 A: id=1을 READY로 UPDATE, publish 시작 (지연 발생)
T1 B: 새 폴링 사이클에서 id=1을 다시 본다? — status='READY'이므로 정상적으론 SELECT 대상 아님
       하지만 A의 READY UPDATE가 트랜잭션 커밋되기 전이라면 B는 여전히 status='NONE'으로 본다 (이슈 2.1과 동일)
```

JPA `save()` 한 건이 짧은 트랜잭션이긴 하지만, `Propagation.SUPPORTS`로 트랜잭션 없이 열리면 가시성 보장이 약하다.
`nextBatchReady`도 `@Transactional(readOnly=true, propagation=SUPPORTS)`라 자체 트랜잭션을 강제하지 않는다.

### 2.3 인스턴스 종료 중 좀비 row

Cloud Run은 트래픽이 줄거나 revision이 바뀌면 인스턴스를 종료한다 (SIGTERM 후 grace period ~10초).

- A 인스턴스가 `READY`로 마킹하고 publish 도중 SIGTERM → `SENT`/`FAILED`로 전이하지 못함
- 다음 폴링 주기부터 `status='READY'`로 영구히 남아 **누구도 재처리하지 않는 좀비**가 됨
- 현재 코드의 `nextBatchReady`는 `status='NONE'`만 조회하므로 `READY` 상태 row는 복구 대상이 아님

### 2.4 At-least-once vs Exactly-once

코드에 이미 다음 TODO가 있다 (`OutboxWorker.java:215-217`):

```java
// TODO
//  - At-least-once 보장을 위해 idempotency key 사용
pubSubPublisherTemplate.publish(PUBLISH_JOB_INIT_TOPIC, message, event.getHeader());
```

Outbox 패턴 자체가 At-least-once를 전제로 하기 때문에 **중복은 다운스트림에서 멱등 처리**하는 것이 정석이지만,
현재는 publish 메시지에 멱등 키가 부여되지 않아 다운스트림이 중복을 식별할 방법이 없다.

## 3. 대응 전략

옵션은 크게 두 축으로 나뉜다: **(a) 중복을 막는다**, **(b) 중복을 허용하되 멱등으로 흡수한다**. 둘 다 병행하는 것이 안전하다.

### 3.1 [권장] DB row-level claim — `SELECT ... FOR UPDATE SKIP LOCKED`

MySQL 8.0+는 `SKIP LOCKED`를 지원한다.

```sql
SELECT id, payload, header, topic
  FROM outbox_events
 WHERE status = 'NONE'
   AND published_time <= NOW()
 ORDER BY id ASC
 LIMIT 10
   FOR UPDATE SKIP LOCKED;
```

- 트랜잭션 내에서 lock을 획득한 row만 가져오고, 다른 트랜잭션이 이미 잠근 row는 건너뛴다.
- **인스턴스가 N개여도 같은 row를 동시에 처리하지 않음**이 DB 레벨에서 보장된다.
- 트랜잭션 안에서 `status='READY'`로 UPDATE까지 마치고 커밋하면 다른 인스턴스의 다음 SELECT 대상에서 제외된다.

적용 시 변경 포인트:
- `OutboxEventJpaRepository.nextBatchReady`에 `@Lock(LockModeType.PESSIMISTIC_WRITE)` + native query로 `FOR UPDATE SKIP LOCKED` 추가
- `getNextBatch`를 `readOnly=false`, `REQUIRES_NEW` 트랜잭션으로 변경
- SELECT → `status='READY'` UPDATE를 같은 트랜잭션에서 처리 후 커밋, 이후 publish는 트랜잭션 밖에서 수행

### 3.2 Claim 컬럼 + Optimistic Update

`outbox_events`에 다음 컬럼 추가:
- `worker_id VARCHAR(64)` — 점유한 인스턴스 식별자
- `claimed_at DATETIME` — 점유 시각
- `version INT` — 낙관적 락

처리 흐름:
```sql
UPDATE outbox_events
   SET status='READY', worker_id=:wid, claimed_at=NOW(), version=version+1
 WHERE id IN (:ids)
   AND status='NONE'
   AND version=:expectedVersion;
```

- 영향받은 행 수(`affected rows`)가 본인이 처리할 대상.
- SKIP LOCKED를 못 쓰는 환경(MySQL 5.7 등)에서 대안으로 유효.
- `claimed_at`이 임계 시간 이상 오래된 row는 좀비로 간주해 reaper가 `NONE`으로 되돌릴 수 있다.

### 3.3 좀비 복구 (Reaper)

- 별도 스케줄러 또는 OutboxWorker 폴링 사이클에서:
  ```sql
  UPDATE outbox_events
     SET status='NONE'
   WHERE status='READY'
     AND updated_time < NOW() - INTERVAL 5 MINUTE;
  ```
- Cloud Run SIGTERM 도중 멈춘 row를 재처리 가능 상태로 되돌린다.
- 임계 시간은 publish 지연 + 안전 마진. 너무 짧으면 정상 처리 중인 row를 빼앗을 수 있으니 보수적으로 설정.

### 3.4 Idempotency Key + 다운스트림 멱등 처리

- Pub/Sub publish 시 message attribute에 `outbox_event_id` (혹은 별도 UUID) 추가
- `JobInitializer` 등 컨슈머가 `processed_event_id` 테이블 또는 Redis 키로 중복 체크
- **3.1/3.2가 누수가 있더라도 최종 안전망**으로 동작 → Outbox 운영의 사실상 표준

### 3.5 Single-writer 강제 (단기 회피책)

- Outbox worker가 동작하는 Cloud Run 서비스에 한해 `max-instances=1` 설정
- 또는 outbox worker를 본 API 서버와 분리된 Cloud Run Job / GKE Deployment(replicas=1)로 빼낸다
- **트래픽 처리 인스턴스와 outbox 인스턴스를 분리**하는 것이 깔끔
- 단점: 처리량 한계, 단일 실패 지점 발생 → 어디까지나 임시 대응 또는 작은 부하 한정

### 3.6 Leader Election (외부 락)

- Redis (`SET NX EX`), Cloud Storage object lock, Spanner row lock 등으로 리더만 outbox를 처리
- 인프라 추가 비용 발생. 위 3.1/3.2로 충분히 커버되므로 우선순위 낮음

## 4. 권장안 (단계별)

| 단계 | 작업 | 효과 | 비용 |
|------|------|------|------|
| 1 | `outbox_events`에 `worker_id`, `claimed_at` 컬럼 추가 (또는 MySQL 8 확인) | 데이터 모델 준비 | 낮음 |
| 2 | `nextBatchReady`를 `SELECT ... FOR UPDATE SKIP LOCKED` + claim UPDATE로 전환 | **중복 publish 차단** | 중 |
| 3 | publish 메시지에 `outbox_event_id` attribute 부여, 다운스트림 멱등 체크 추가 | 안전망 | 중 |
| 4 | `READY` 좀비 reaper 추가 (5분 임계) | SIGTERM 손실 복구 | 낮음 |
| 5 | (선택) outbox worker만 별도 서비스로 분리, `max-instances=1` 검토 | 인프라 단순화 | 중~상 |

`synchronized(lock)`은 단일 인스턴스 가정의 부산물이므로, 위 변경 후에는 의미가 없어진다. (그대로 둬도 무해하지만 멀티스레드 처리를 막는 부작용이 있으니 ExecutorService 활용과 함께 재검토 필요.)

## 5. 참고

- 코드:
  - `src/main/java/com/kakaohealthcare/mydata/event/worker/OutboxWorker.java`
  - `src/main/java/com/kakaohealthcare/mydata/event/OutboxEventJpaRepository.java`
  - `src/main/java/com/kakaohealthcare/mydata/event/model/OutboxEvent.java`
  - `src/main/java/com/kakaohealthcare/mydata/OutboxInitializer.java`
- 외부 자료:
  - GCP Cloud Run: [Container instance scaling](https://cloud.google.com/run/docs/about-instance-autoscaling)
  - MySQL: [SELECT ... FOR UPDATE SKIP LOCKED](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)
  - Microservices.io: [Transactional Outbox Pattern](https://microservices.io/patterns/data/transactional-outbox.html)
