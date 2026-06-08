# Outbox 좀비 Reaper — 분석/설계

> [outbox-concurrency](./outbox-concurrency.md) 문서의 **3.3 좀비 복구** 항목을 단독 설계한다.
> Redis 외부 락(3.6)을 채택하더라도 **반드시 병행**해야 하는 안전망이다.

## 1. 좀비란 무엇인가

`outbox_events` 테이블에서 **`status = 'READY'` 인 채로 영구히 멈춰버린 row**.

`OutboxWorker.processEvent`는 다음 순서로 동작한다:

```
1. status NONE → READY  (UPDATE)
2. Pub/Sub publish
3. status READY → SENT  (UPDATE)
```

2번 도중 (또는 1번 직후 ~ 2번 시작 전) **인스턴스가 죽으면** 3번이 실행되지 않는다.
다음 leader는 `nextBatchReady`로 `status='NONE'` 만 조회하므로 **READY 좀비는 영원히 방치**된다.

## 2. 좀비 발생 시나리오

| 시나리오 | 발생 메커니즘 |
|----------|---------------|
| **SIGTERM grace period 초과** | Cloud Run 10초 제한. publish가 8초+ 걸리면 미완료 상태로 종료 |
| **OOM Kill / kill -9** | `@PreDestroy` 자체가 호출되지 않음 → 락도 안 풀리고 row도 READY로 남음 |
| **JVM crash** | 동상 |
| **DB connection 일시 단절** | `READY` UPDATE는 성공, publish는 성공, `SENT` UPDATE 직전 connection drop |
| **Pub/Sub publish 타임아웃** | `processEvent`의 catch에서 FAILED로 처리되지만, catch 자체가 호출되기 전 인스턴스 종료되면 READY |
| **Reaper 자체 버그** | (메타) reaper가 NONE으로 잘못 되돌린 row가 다시 READY로 진입하는 무한 루프 |

**외부 락이 있어도 좀비는 발생한다**: 외부 락은 "동시에 누가 처리하느냐"만 보장할 뿐, **죽음 직전 미완료 상태**를 막아주지는 못한다.

## 3. Reaper 동작 설계

### 3.1 기본 쿼리

```sql
UPDATE outbox_events
   SET status = 'NONE',
       updated_time = NOW()
 WHERE status = 'READY'
   AND updated_time < NOW() - INTERVAL :threshold_minute MINUTE;
```

영향받은 행 수 = 복구된 좀비 수 → 메트릭 노출.

### 3.2 임계 시간 (`threshold_minute`)

- 너무 짧으면: 정상 처리 중인 row를 빼앗아 **중복 publish 유발**
- 너무 길면: 좀비 잔류 → 사용자 데이터 수집 지연

**기준 산정:**
- Pub/Sub publish p99 응답시간 (보통 < 1초)
- + Cloud Run SIGTERM grace period (10초)
- + 안전 마진 (× 10~30 배)
- = **5분** 권장 (보수적). 운영 데이터 보고 조정.

### 3.3 재시도 횟수 제한

좀비를 무한히 NONE으로 되돌리면 publish가 영원히 실패하는 row가 매번 부활한다. 방지:

```sql
UPDATE outbox_events
   SET status = CASE
       WHEN retry_count >= 5 THEN 'FAILED'
       ELSE 'NONE'
   END,
       retry_count = retry_count + 1,
       updated_time = NOW()
 WHERE status = 'READY'
   AND updated_time < NOW() - INTERVAL 5 MINUTE;
```

- 5회 초과 → `FAILED`로 격리. 운영자가 별도 조사.
- `OutboxEvent.retryCount` 컬럼은 이미 존재 (현재 미사용)

## 4. 구현 위치 옵션

| 옵션 | 위치 | 장점 | 단점 |
|------|------|------|------|
| A. **OutboxWorker 폴링 사이클 내** | `run()` 매 N회 사이클마다 reaper 1회 | 가장 단순, 추가 빈 불필요 | leader만 실행 → leader 죽으면 reaper도 정지 |
| B. **별도 `@Scheduled` Reaper 빈** | `OutboxReaperScheduler.runReaper()` | 책임 분리, 테스트 용이 | leader 가드 필요 (외부 락 체크) |
| C. **DB 트리거/이벤트** | MySQL Event Scheduler | 앱 외부에서 동작, 인스턴스 무관 | 운영 가시성 ↓, DB 관리 부담 |

**권장: B (별도 `@Scheduled` Reaper 빈)**
- leader 가드와 결합:
  ```java
  @Scheduled(fixedDelay = 60_000)  // 1분마다
  public void runReaper() {
      if (!leaderElector.isLeader()) return;  // leader만 실행
      int reaped = outboxReaperRepository.reapZombies(Duration.ofMinutes(5));
      meterRegistry.counter("outbox.reaper.reaped").increment(reaped);
  }
  ```
- leader가 reaper를 책임지는 모델로 단일 인스턴스 보장

## 5. 상태 머신 보완

```
        ┌────────────────────────────────────┐
        │                                    │
        ▼                                    │
  ┌──────────┐  pick   ┌─────────┐  publish ┌────────┐
  │   NONE   │────────►│  READY  │─────────►│  SENT  │
  └──────────┘         └────┬────┘          └────────┘
        ▲                   │
        │                   │ 5분 초과 + retry_count < 5
        └───────────────────┘
                            │
                            │ 5분 초과 + retry_count ≥ 5
                            ▼
                       ┌─────────┐
                       │ FAILED  │
                       └─────────┘
```

## 6. 모니터링

| 메트릭 | 의미 | 알람 임계 |
|--------|------|-----------|
| `outbox.reaper.reaped` (counter) | 복구된 좀비 수 | 시간당 > 10 이면 SIGTERM 처리 시간 부족 의심 |
| `outbox.reaper.failed_promoted` (counter) | retry 한계 초과로 FAILED 격리된 수 | > 0 즉시 조사 |
| `outbox.reaper.last_run_at` (gauge) | 마지막 실행 시각 | 10분 이상 미실행 시 알람 |

## 7. 운영 가시성

### 7.1 현재 좀비 수 확인

```sql
SELECT COUNT(*), MIN(updated_time)
  FROM outbox_events
 WHERE status = 'READY'
   AND updated_time < NOW() - INTERVAL 5 MINUTE;
```

### 7.2 좀비 강제 복구

```sql
UPDATE outbox_events SET status='NONE' WHERE id IN (...);
```

### 7.3 FAILED 격리 row 재처리

운영자 판단 후:
```sql
UPDATE outbox_events SET status='NONE', retry_count=0 WHERE id = ?;
```

## 8. 다른 옵션과의 관계

| 옵션 | Reaper와의 관계 |
|------|----------------|
| 3.1 비관적 락 (`SKIP LOCKED`) | **병행** — 락이 있어도 SIGTERM 좀비는 발생 |
| 3.2 낙관적 락 | **병행** — 동상 |
| 3.4 Idempotency Key | **병행 강력 권장** — reaper가 NONE으로 되돌린 row는 재발행되므로 다운스트림 중복 흡수 필요 |
| 3.6 외부 락 (Redis) | **병행 필수** — 본 문서가 그 짝 |

## 9. 결론

- 외부 락 채택 후에도 **READY 좀비는 반드시 발생**한다 (SIGTERM, OOM, network drop)
- Reaper는 **5분 임계 + retry 5회 제한**으로 가볍게 구현 가능
- `@Scheduled` + leader 가드 조합이 가장 깔끔
- **Idempotency Key(3.4)와 반드시 함께** 채택해야 안전 — reaper가 부활시킨 row는 두 번 publish 가능
