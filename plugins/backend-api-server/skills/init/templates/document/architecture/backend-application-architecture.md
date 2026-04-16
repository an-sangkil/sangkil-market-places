# Backend Application Architecture

## 목적

- 백엔드 기능 개발 시 공통 구조를 통일한다.
- 기능별 planning 문서는 본 문서를 참조하고, 기능 특화 내용만 작성한다.
- 구현 시작 전 본 문서를 확인하는 것을 기본 절차로 한다.

## 기본 레이어 원칙

- `Controller`
  - HTTP 요청/응답, 인증 사용자 컨텍스트, 파라미터 검증 담당
  - 비즈니스 규칙 계산은 하지 않는다.
- `Service`
  - 도메인 규칙/조합/정렬/필터링 담당
  - DTO 조립 책임을 가진다.
- `Repository`
  - DB 조회/저장 책임 담당
  - 조회 패턴은 용도별 메서드로 분리한다.

## 모델 분리 원칙

- `Entity (DB)`
  - 영속 모델, 테이블/컬럼 매핑 책임
  - 네임스페이스: `model/entity/{category}/...`
- `Request DTO`
  - API 입력 모델
  - 네임스페이스: `model/request/{domain}/...`
- `Response DTO`
  - API 출력 모델
  - 네임스페이스: `model/response/{domain}/...`
- `Record/Projection`
  - 서비스 내부 조합 또는 조회 Projection 전용 모델
  - 네임스페이스: 서비스 패키지 또는 `repository/mybatis/dto`

## 패키지 컨벤션

- API 컨트롤러: `api/{domain}/*Controller`
- 서비스: `api/{domain}/service/*`
- JPA 리포지토리: `repository/jpa/{category}/*`
- MyBatis 매퍼: `repository/mybatis/*`

> 프로젝트가 멀티 모듈 구조라면 위 네임스페이스를 각 모듈에 매핑하여 사용한다.
> (예: 모델은 `model` 모듈, Repository는 `infrastructure` 모듈)
> 구체 매핑은 프로젝트 `CLAUDE.md`에 명시한다.

## Repository 생성/네이밍 규칙

- JPA Repository 클래스명은 테이블명을 기준으로 생성한다.
- 규칙: `snake_case` 테이블명을 `PascalCase`로 변환한 뒤 `JpaRepository` 접미사를 붙인다.
- 예시: `user_account` 테이블은 `UserAccountJpaRepository`로 생성한다.

## 개발 절차(필수 체크)

1. 기능 planning 문서에서 요구사항/비범위/리스크를 확인한다.
2. 본 문서 구조를 기준으로 Controller/Service/Repository 책임을 먼저 나눈다.
3. Entity와 Request/Response DTO를 분리 설계한다.
4. Repository 쿼리와 Service 규칙(정렬/필터/검증)을 분리 구현한다.
5. 컨트롤러는 입력 검증과 결과 포맷만 담당하도록 확인한다.
6. 테스트(서비스 단위 + 컨트롤러 통합)와 문서를 동기화한다.

## 기능 문서 작성 규칙

- 기능별 실행계획 문서에는 아래 문구를 포함한다.
  - `공통 아키텍처: document/architecture/backend-application-architecture.md`
- 기능 문서에 공통 구조를 중복 서술하지 않고, 본 문서 링크로 대체한다.
