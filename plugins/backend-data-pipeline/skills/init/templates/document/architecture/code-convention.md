# 코드 컨벤션 — 패키지 구조 및 네이밍 가이드

## 1) 패키지 구조

```
{root-package}/
│
├── aspect/                          # AOP 관심사 (트랜잭션, MDC 등)
├── collect/                         # ⭐ 핵심 도메인: 데이터 수집 파이프라인
│   ├── consts/                      #   수집 관련 상수/열거형
│   ├── core/                        #   Job 오케스트레이션
│   │   ├── factory/                 #     수집 대상 생성 팩토리
│   │   └── pool/                    #     실행 전략 (Sync/Async)
│   ├── metrics/                     #   수집 모니터링/메트릭
│   ├── model/                       #   파싱 결과 DTO (Record)
│   │   └── entity/                  #   JPA Entity + 복합키
│   ├── parser/                      #   원시 데이터 → Record 변환
│   ├── saver/                       #   Record → DB 저장
│   ├── target/                      #   수집 대상 결정
│   └── v{N}/crawler/                #   버전별 크롤러
│
├── config/                          # Spring 설정 (@Configuration)
├── controller/                      # REST 컨트롤러
│
├── event/                           # 이벤트 처리 (Outbox 패턴 등)
│   ├── model/                       #   이벤트 모델
│   └── worker/                      #   이벤트 소비 워커
│
├── model/                           # 공통 요청/응답 모델
├── mq/                              # 메시지 큐 (Pub/Sub, Kafka 등)
│   ├── {domain}/
│   │   ├── dto/
│   │   ├── publish/
│   │   └── subscribe/
│   └── utils/                       #   트레이싱/MDC 유틸
│
└── repository/                      # JPA Repository
```

> **NOTE:** 위 구조를 프로젝트에 맞게 커스터마이즈한다. 루트 패키지와 도메인별 세부 패키지는 프로젝트마다 다를 수 있다.

## 2) 클래스 네이밍 규칙

### 2.1 모델 네이밍

모든 모델 클래스는 **용도를 접미사로 명확히 구분**한다.

| 패턴 | 용도 | 예시 |
|------|------|------|
| `{기능}Record` | 내부 데이터 전달 (파싱 결과 등) | `DiagnosticObservationRecord` |
| `{기능}Request` | 외부 요청 DTO | `CollectRequest` |
| `{기능}Response` | 외부 응답 DTO | `DiagnosticResponse` |
| `{테이블명}Entity` | JPA 엔티티 (DB 매핑) | `DiagnosticObservationEntity` |
| `{테이블명}Id` | 복합 PK 클래스 (`@EmbeddedId`) | `DiagnosticObservationId` |

### 2.2 접미사 (역할별)

| 접미사 | 역할 | 위치 |
|--------|------|------|
| `*Record` | 내부 DTO (파싱 결과 등) | `collect/model/` |
| `*Request` | 외부 요청 DTO | `model/` |
| `*Response` | 외부 응답 DTO | `model/` |
| `*Entity` | JPA 엔티티 | `collect/model/entity/` |
| `*Id` | 복합 PK 클래스 | `collect/model/entity/` |
| `*Parser` | 원시 데이터 → Record 변환기 | `collect/parser/` |
| `*Saver` | Record → DB 저장 로직 | `collect/saver/` |
| `*Crawler` | 수집 흐름 조율 (API 호출 → 파싱 → 저장) | `collect/v{N}/crawler/` |
| `*Collector` | 수집 실행 베이스 클래스 | `collect/v{N}/crawler/` |
| `*Strategy` | 실행 전략 (Strategy 패턴) | `collect/core/pool/` |
| `*Factory` | 객체 생성 팩토리 | `collect/core/factory/` |
| `*Subscriber` | 메시지 수신 핸들러 | `mq/*/subscribe/` |
| `*PublisherService` | 메시지 발행 서비스 | `mq/*/publish/` |
| `*Repository` | Spring Data JPA Repository | `repository/` |
| `*Config` | Spring `@Configuration` 클래스 | `config/` |
| `*Properties` | `@ConfigurationProperties` 바인딩 | `config/` |
| `*Manager` | 상태/리소스 관리자 | 해당 도메인 패키지 |

## 3) 신규 코드 작성 가이드

### 3.1 패키지 배치 원칙

- **수집 파이프라인 관련 코드**는 반드시 `collect/` 하위에 둔다
- **Parser, Saver는 분리**한다 — 파싱과 저장 책임을 한 클래스에 섞지 않는다
- **Record ↔ Entity 변환**은 `Record.toEntity()`에서 수행한다
- **메시지 큐 관련 코드**는 `mq/{도메인}/publish|subscribe/` 구조를 따른다
- **JPA Repository**는 `repository/`에 모은다

### 3.2 버저닝

- 크롤러의 대규모 구조 변경 시 `v{N}/crawler/` 패키지로 버전을 분리한다
- Parser, Saver, Record, Entity는 버저닝하지 않는다 (스키마 변경은 마이그레이션으로 처리)

### 3.3 네이밍 체크리스트

신규 클래스 작성 시:

- [ ] 역할에 맞는 접미사를 사용했는가? (`*Record`, `*Entity`, `*Parser` 등)
- [ ] 도메인 구분이 필요한 경우 접두사를 붙였는가?
- [ ] 베이스 클래스는 `Abstract*`로 시작하는가?
- [ ] 복합키 클래스는 `*Id`로 끝나는가?
- [ ] Repository 이름에 `Jpa`를 포함했는가? (`*JpaRepository`)

### 3.4 테스트 패키지

- 테스트 클래스는 **대상 클래스와 동일한 패키지 경로**에 둔다
- 네이밍: `{대상클래스명}Test.java`

### 3.5 테스트 전략: Mock + Fixture 기반

| 유형 | 방식 | DB 사용 | 사용 시점 |
|------|------|---------|-----------|
| **단위 테스트 (기본)** | Mock + Fixture | X | Parser, Saver, Service 로직 검증 |
| **통합 테스트 (명시 요청 시)** | 실제 DB + Fixture | O | 사용자가 실제 DB 연동을 요청한 경우 |

- 외부 의존성(Repository, SDK, MQ)은 **Mockito로 Mock** 처리
- 테스트 데이터는 **Fixture Enum/Factory**에서 가져온다
- DAMP 원칙: 테스트 코드는 중복 허용 (가독성 > DRY)
