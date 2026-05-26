# 코드 컨벤션 — 패키지 구조 및 네이밍 가이드

## 1) 패키지 구조

```
{root-package}/
│
├── api/                             # 도메인별 Controller + Service
│   └── {domain}/                    #   도메인 디렉토리
│       ├── controller/              #     REST 컨트롤러
│       └── service/                 #     비즈니스 서비스
│
├── audit/                           # 감사 로그
├── auth/                            # 인증 필터
├── cache/                           # 캐시 전략
│
├── collect/                         # 데이터 수집 파이프라인
│   ├── core/                        #   TargetGeneration, JobInitializer
│   ├── crawler/                     #   수집 크롤러
│   ├── parser/                      #   수집 데이터 파싱
│   └── model/                       #   수집 전용 모델
│
├── config/                          # 글로벌 설정 (@Configuration)
├── errorlog/                        # 에러 로깅
├── schedule/                        # ShedLock 기반 스케줄러
├── statistics/                      # 통계
└── aop/                             # 횡단 관심사
```

> **NOTE:** 위 구조를 프로젝트에 맞게 커스터마이즈한다. 멀티 모듈 구조인 경우 각 모듈에 매핑하여 사용한다.

## 2) 클래스 네이밍 규칙

### 2.1 모델 네이밍

모든 모델 클래스는 **용도를 접미사로 명확히 구분**한다.

| 패턴 | 용도 | 예시 |
|------|------|------|
| `{기능}Request` | API 요청 DTO | `DiagnosticReportRequest` |
| `{기능}Response` | API 응답 DTO | `DiagnosticReportResponse` |
| `{테이블명}` | JPA 엔티티 (DB 매핑) | `DiagnosticObservation` |
| `{기능}VO` | 값 객체 | `DiagnosticRiskLevelVO` |

### 2.2 접미사 (역할별)

| 접미사 | 역할 | 위치 |
|--------|------|------|
| `*Request` | API 요청 DTO | `model/request/{domain}/` |
| `*Response` | API 응답 DTO | `model/response/{domain}/` |
| `*VO` | 값 객체 | `model/vo/` 또는 서비스 패키지 |
| `*Controller` | REST 컨트롤러 | `api/{domain}/controller/` |
| `*Service` | 비즈니스 서비스 | `api/{domain}/service/` |
| `*JpaRepository` | Spring Data JPA Repository | `repository/jpa/{category}/` |
| `*Mapper` | MyBatis Mapper | `repository/mybatis/` |
| `*Config` | Spring `@Configuration` | `config/` |
| `*Properties` | `@ConfigurationProperties` | `config/` |
| `*Crawler` | 수집 크롤러 | `collect/crawler/` |
| `*Parser` | 수집 데이터 파서 | `collect/parser/` |
| `*Scheduler` | ShedLock 스케줄러 | `schedule/` |

## 3) 신규 코드 작성 가이드

### 3.1 패키지 배치 원칙

- **API 관련 코드**는 `api/{domain}/` 하위에 Controller와 Service를 함께 둔다
- **엔티티**는 `model/entity/{category}/`에 카테고리별로 분리한다
- **Request/Response DTO**는 `model/request/{domain}/`, `model/response/{domain}/`에 둔다
- **JPA Repository**는 `repository/jpa/{category}/`에 모은다
- **수집 파이프라인 관련 코드**는 `collect/` 하위에 둔다

### 3.2 Repository 생성 규칙

- JPA Repository 클래스명은 테이블명을 기준으로 생성한다
- 규칙: `snake_case` 테이블명을 `PascalCase`로 변환한 뒤 `JpaRepository` 접미사
- 예시: `user_account` 테이블 -> `UserAccountJpaRepository`

### 3.3 네이밍 체크리스트

신규 클래스 작성 시:

- [ ] 역할에 맞는 접미사를 사용했는가? (`*Service`, `*Controller`, `*JpaRepository` 등)
- [ ] 도메인 구분이 필요한 경우 접두사를 붙였는가?
- [ ] Repository 이름에 `Jpa`를 포함했는가?

### 3.4 테스트 패키지

- 테스트 클래스는 **대상 클래스와 동일한 패키지 경로**에 둔다
- 네이밍: `{대상클래스명}Test.java`

### 3.5 테스트 전략: Mock + Fixture 기반

| 유형 | 방식 | DB 사용 | 사용 시점 |
|------|------|---------|-----------|
| **단위 테스트 (기본)** | Mock + Fixture | X | Service 로직, Parser 검증 |
| **통합 테스트 (명시 요청 시)** | TestContainers/H2 + Fixture | O | DB 연동 필요 시 |

- 외부 의존성(Repository, SDK, Feign)은 **Mockito로 Mock** 처리
- 테스트 데이터는 `mydata-model/src/testFixtures/` 의 Fixture 활용
- DAMP 원칙: 테스트 코드는 중복 허용 (가독성 > DRY)
