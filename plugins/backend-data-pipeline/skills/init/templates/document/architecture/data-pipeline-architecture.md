# 데이터 파이프라인 아키텍처

## 목적

- 이벤트 드리븐 데이터 수집 파이프라인의 공통 구조를 정의한다.
- 기능별 planning 문서는 본 문서를 참조하고, 기능 특화 내용만 작성한다.
- 구현 시작 전 본 문서를 확인하는 것을 기본 절차로 한다.

## 파이프라인 5단계 구조

```
메시지 수신 → Job 초기화 → 데이터 수집 → 데이터 파싱 → 데이터 저장
```

### 1단계: 메시지 구독

- 메시지 큐(Pub/Sub, Kafka 등)에서 수집 요청 메시지 수신
- Ack/Nack 및 재시도 로직 처리
- 수신한 메시지를 Job 초기화 단계로 위임

### 2단계: Job 오케스트레이션

- 수집 요청을 Job 객체로 변환
- Job 계층: 마스터 Job → 하위 Job
- 상태 추적: NONE → RUNNING → COMPLETE/ERROR/FAIL
- TargetGeneration: 수집 요청에서 적절한 Job 인스턴스 생성

### 3단계: 데이터 수집

- Collector/Crawler가 외부 데이터 소스에서 원시 데이터 수집
- 서로 다른 데이터 소스별 수집 전략 구현
- 버전별 수집기 관리 (v1/v2 등)

### 4단계: 데이터 파싱

- 수집한 원시 데이터를 도메인 Record로 변환
- Parser별 책임 분리 (데이터 유형별 Parser)
- 파싱 결과는 Record DTO에 담는다

### 5단계: 데이터 저장

- Record → Entity 변환 후 DB 저장
- Saver가 Repository를 이용해 저장/갱신
- 1:N 관계 처리 (부모 먼저, 자식 나중)
- 트랜잭션 경계 관리

## 주요 디자인 패턴

- **팩토리 패턴**: TargetGeneration이 적절한 Job 인스턴스 생성
- **템플릿 메서드**: AbstractJob이 Job 생명주기를 정의하고 하위 클래스가 구체 내용 구현
- **전략 패턴**: 다양한 데이터 소스를 위한 서로 다른 Crawler와 Parser
- **옵저버 패턴**: Job 진행 상태 추적 및 이벤트 발행

## 데이터 흐름

```
원시 데이터 (Bundle/JSON/...)
    │
    ▼
  *Parser        — 원시 데이터 → *Record 변환
    │
    ▼
  *Record        — 파싱 결과 DTO (중간 모델)
    │  .toEntity()
    ▼
  *Entity        — JPA 엔티티 (DB 매핑)
    │
    ▼
  *Saver         — *Repository를 이용한 저장/갱신
    │
    ▼
  *JpaRepository — Spring Data JPA
```

## 신규 파이프라인 추가 시 필요한 클래스

| 순서 | 클래스 | 패키지 | 역할 |
|------|--------|--------|------|
| 1 | `{접두사}{리소스}Record` | `collect/model/` | 파싱 결과 DTO |
| 2 | `{접두사}{리소스}Entity` (+`*Id` if 복합키) | `collect/model/entity/` | DB 매핑 |
| 3 | `{접두사}{리소스}Parser` | `collect/parser/` | 원시 데이터 → Record 변환 |
| 4 | `{접두사}{리소스}Saver` | `collect/saver/` | Record → DB 저장 |
| 5 | `{접두사}{리소스}JpaRepository` | `repository/` | Spring Data JPA |

## 기능 문서 작성 규칙

- 기능별 실행계획 문서에는 아래 문구를 포함한다.
  - `공통 아키텍처: docs/architecture/data-pipeline-architecture.md`
- 기능 문서에 공통 구조를 중복 서술하지 않고, 본 문서 링크로 대체한다.
