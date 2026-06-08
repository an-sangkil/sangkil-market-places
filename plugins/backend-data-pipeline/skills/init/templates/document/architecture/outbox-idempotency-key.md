# Outbox Idempotency Key — 분석/설계

> [outbox-concurrency](./outbox-concurrency.md) 문서의 **3.4 멱등성 처리** 항목을 단독 설계한다.
> 외부 락(3.6) 채택 후에도 **병행 권장**되는 최종 안전망. Outbox 패턴의 사실상 표준.

## 1. 왜 필요한가

Outbox는 본질적으로 **At-least-once** 보장 패턴이다. Exactly-once는 패턴 자체로는 불가능하고, **다운스트림에서 멱등 처리**로 달성한다.

중복이 발생할 수 있는 지점:

| 지점 | 메커니즘 |
|------|----------|
| 외부 락 흔들림 | Leader 교체 직후 짧은 윈도우에서 두 인스턴스가 같은 row를 publish |
| Memorystore 페일오버 | 페일오버 중 락이 잠시 분실 → 다중 leader 가능 |
| **Reaper의 부활** | `READY` 좀비를 `NONE`으로 되돌렸는데, 사실 원래 leader가 publish는 성공했고 SENT UPDATE만 실패한 경우 → 새 leader가 다시 publish |
| Pub/Sub 자체 중복 | Pub/Sub는 at-least-once 전제. ack 실패 시 동일 메시지 재전송 |
| Subscriber 재시작 | 컨슈머가 처리 중 죽어 ack 못 보내면 Pub/Sub가 다시 전달 |

**현재 상태:**

`OutboxWorker.publishEvent`에 이미 다음 TODO가 있다:
```java
// TODO
//  - At-least-once 보장을 위해 idempotency key 사용
pubSubPublisherTemplate.publish(PUBLISH_JOB_INIT_TOPIC, message, event.getHeader());
```
**미부여 → 다운스트림이 중복을 식별할 방법 없음.**

## 2. Idempotency Key 설계

### 2.1 Key 값

| 후보 | 평가 |
|------|------|
| **`outbox_event_id` (Long)** | DB PK, 자연스럽게 unique, 별도 생성 불필요 [권장] |
| UUID | 충돌 없음, 가독성 ↓, 별도 컬럼 필요 |
| `userId + functionType + timestamp` 조합 | 비즈니스 의미 있지만 milli 단위 충돌 가능, 비추 |

**선정: `outbox_event_id`**. 이미 존재하는 PK이므로 publish 시점에 추가 작업 0.

### 2.2 전달 위치

Pub/Sub message attribute:
```java
Map<String, String> attributes = new HashMap<>(event.getHeader());
attributes.put("idempotency_key", String.valueOf(event.getId()));
pubSubPublisherTemplate.publish(topic, message, attributes);
```

**왜 attribute인가:**
- 메시지 payload를 건드리지 않음 (기존 컨슈머 호환)
- Pub/Sub attribute는 키 단위 노출이 쉬워 컨슈머에서 추출 비용 ↓
- ordering_key와 별개로 운영 가능

### 2.3 발급 시점

- **publish 직전**이 정답. outbox row 생성 시점이 아님.
- row 생성 시점에 발급하면 retry 시 같은 key로 재발행되어 중복 제거가 의도대로 작동
- (`outbox_event_id` 자체가 row 생성 시 확정되므로 위 정의와 부합)

## 3. 다운스트림 중복 제거 옵션

### 3.1 DB 테이블 방식 [권장]

```sql
CREATE TABLE processed_outbox_events (
    idempotency_key VARCHAR(64) PRIMARY KEY,
    consumer_name   VARCHAR(64) NOT NULL,
    processed_at    DATETIME(3) NOT NULL,
    INDEX idx_processed_at (processed_at)
);
```

처리 패턴:
```java
@Transactional
public void handle(MedicalPublishRequest req, String idempotencyKey) {
    int inserted = processedRepo.insertIfAbsent(idempotencyKey, "JobInitializer");
    if (inserted == 0) {
        log.info("duplicate skipped: key={}", idempotencyKey);
        return;  // ACK만 보내고 종료
    }
    // 비즈니스 로직 (JobManager 등록, 다운스트림 호출 등)
}
```

- `INSERT IGNORE` 또는 `INSERT ... ON DUPLICATE KEY UPDATE id=id` (no-op)
- 영향 행수 = 0 → 중복, = 1 → 신규
- **같은 트랜잭션에서 INSERT + 비즈니스 처리** → 원자성 보장

**TTL 관리:**
- `processed_at` 기준 7일 지난 row 삭제 (스케줄러)
- Pub/Sub redelivery window보다 길게 (Pub/Sub 최대 7일)

### 3.2 Redis SETNX 방식 (대안)

```java
Boolean acquired = redis.opsForValue()
    .setIfAbsent("processed:" + idempotencyKey, "1", Duration.ofDays(7));
if (Boolean.FALSE.equals(acquired)) return;
```

장점: 빠름, TTL 자동
단점: Redis 장애 시 멱등성 가드 자체가 무력화 → DB 백업 필요. 분산 환경에서 INSERT 트랜잭션과 비즈니스 로직 사이 원자성 깨질 위험

### 3.3 Bloom Filter (대용량 한정, 미추천)

- 메모리 효율 ↑, false positive 0이 아님 → 정합성 손해 가능
- 의료 데이터 도메인에선 추천 안 함

### 옵션 비교

| 항목 | DB 테이블 | Redis SETNX | Bloom |
|------|----------|-------------|-------|
| 정확성 | 100% | 100% (Redis 살아있을 때) | False positive 발생 |
| 비즈니스 로직과 원자성 | ◎ (같은 트랜잭션) | △ (분산 트랜잭션 어려움) | × |
| 의존성 | 기존 MySQL | Redis 추가 (이미 외부 락용 있음) | 라이브러리 |
| 운영 가시성 | SELECT 한 줄 | `redis-cli EXISTS` | 거의 없음 |

**선정: 3.1 DB 테이블** (외부 락에 이미 Redis 의존성이 있지만, 멱등성은 비즈니스 로직과 원자적이어야 하므로 같은 DB 트랜잭션이 자연스럽다)

## 4. 구현 위치

### 4.1 Publisher 측 (OutboxWorker)

```java
void publishEvent(OutboxEvent event) {
    String payload = JsonUtil.serialize(event.getPayload());
    MedicalPublishRequest request = JsonUtil.deserialize(payload, MedicalPublishRequest.class);
    List<CollectRequest> collectRequests = targetFactoryService.collectRequests(
        request.getUserId(), request.getFunctionType());
    String message = JsonUtil.serialize(collectRequests);

    Map<String, String> attributes = new HashMap<>(
        Optional.ofNullable(event.getHeader()).orElseGet(HashMap::new));
    attributes.put("idempotency_key", String.valueOf(event.getId()));

    pubSubPublisherTemplate.publish(PUBLISH_JOB_INIT_TOPIC, message, attributes);
}
```

변경 분량: **3줄 추가**. 기존 동작 영향 없음.

### 4.2 Consumer 측 (JobInitializer 등)

- 컨슈머 진입점에서 attribute 추출 → `processed_outbox_events` 체크 → 비즈니스 로직
- 컨슈머별로 `consumer_name`을 다르게 기록하면 **같은 메시지를 여러 컨슈머가 처리**할 때 각자 멱등성 보장 가능

### 4.3 별도 컨슈머 (외부 시스템)

- 외부 시스템이 같은 topic을 구독한다면 그 쪽에서도 같은 attribute를 보고 자체 체크
- 본 프로젝트는 내부 컨슈머만 우선 적용

## 5. 트랜잭션 패턴

**원자성이 핵심**. 중복 체크 INSERT와 비즈니스 로직이 같은 트랜잭션 안에 있어야 한다:

```
@Transactional
processMessage:
  ├─ INSERT IGNORE processed_outbox_events (key)
  ├─ if affected == 0: return  (이미 처리됨)
  ├─ businessLogic()
  └─ commit  ← 비즈니스 실패 시 INSERT도 롤백 → 다음 retry에서 재처리
```

비즈니스 로직이 외부 시스템 호출을 포함하면 **트랜잭션 경계를 어떻게 잡을지 별도 설계 필요** (Outbox 패턴이 본 프로젝트 안에 또 등장하는 재귀 구조 가능).

## 6. TTL 정책

```sql
DELETE FROM processed_outbox_events
 WHERE processed_at < NOW() - INTERVAL 7 DAY;
```

- 일 1회 야간 배치 또는 `@Scheduled`
- **7일**: Pub/Sub redelivery window 최대치와 일치
- 보관 비용 추정: 일 10만 건 × 7일 = 70만 row, key 16바이트 + 메타 50바이트 → ~50MB. 무시 가능 수준

## 7. 모니터링

| 메트릭 | 의미 | 알람 임계 |
|--------|------|-----------|
| `outbox.idempotency.duplicate_skipped` | 중복으로 스킵된 수 | 0이 정상. > 시간당 10이면 외부 락 불안정 의심 |
| `outbox.idempotency.processed` | 정상 처리 수 | 트래픽 모니터링용 |
| `processed_outbox_events.size` | 테이블 크기 | TTL 미작동 감지 |

## 8. 다른 옵션과의 관계

| 옵션 | Idempotency Key와의 관계 |
|------|------------------------|
| 3.1 비관적 락 | **병행 권장** — 락 누수 시 안전망 |
| 3.2 낙관적 락 | **병행 권장** — 동상 |
| 3.3 좀비 Reaper | **병행 필수** — reaper가 부활시킨 row는 중복 publish 가능 |
| 3.6 외부 락 (Redis) | **병행 권장** — leader 교체기 / Memorystore 페일오버 안전망 |
| Pub/Sub redelivery | **항상 병행** — Pub/Sub 자체 at-least-once 보장의 안전망 |

## 9. 결론

- Idempotency Key는 **Outbox 패턴 운영의 사실상 표준**
- 변경 분량 적음: Publisher 3줄 + Consumer 가드 1트랜잭션 + 테이블 1개
- Key 값은 `outbox_event_id` 그대로 사용 (자연 unique)
- 다운스트림 중복 제거는 **DB 테이블 + INSERT IGNORE** 권장 (비즈니스 트랜잭션과 원자성)
- TTL 7일로 충분
- **외부 락 + Reaper + Idempotency Key** 3종 세트가 mydata-collector outbox의 권장 운영 구성
