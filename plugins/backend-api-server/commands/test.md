---
description: 테스트 실행 + 테스트 품질 리뷰 (코드 QA 체크리스트는 스킵)
---

현재 변경사항의 테스트를 실행하고 테스트 품질만 리뷰한다. 코드 QA 체크리스트는 수행하지 않는다.

**요청 내용 (선택):**
$ARGUMENTS

## 실행 방식

### 1) 변경사항 파악
- `git status`, `git diff --name-only HEAD`로 수정/생성된 파일 목록 확인
- `mydata-model/src/testFixtures/`, `*/src/test/java/**` 변경 여부 함께 확인

### 2) 테스트 실행
다음 명령 중 변경 범위에 맞는 것을 실행:

```bash
./gradlew :mydata-model:test
./gradlew :mydata-share-infrastructure:mydata-share-repository:test
./gradlew :karechat-mydata-api:test
```

실패 시 개별 테스트 단위로 재실행하여 실패 원인을 파악.

### 3) test-qa 에이전트 호출

Agent tool로 다음을 전달:
- **focus: `test-only`** — 테스트 실행 결과와 테스트 코드 품질만 리뷰
- 변경된 파일 목록
- 테스트 실행 로그
- 개발 플랜 경로 (있으면)

## 에이전트 동작 (focus=test-only)

- 테스트 실행 결과 집계 (전체/성공/실패/스킵)
- 테스트 품질 리뷰
  - 조건 분기 커버리지
  - DAMP 원칙 준수 여부
  - 상태 기반 vs 상호작용 기반 검증
  - Mock 사용 적절성
- **코드 QA 체크리스트는 스킵** (빠른 테스트 중심 점검)

## 산출물

`docs/output/{domain}/test-report.md` (해당 도메인이 파악되는 경우)

## 결과 보고

테스트 통과 여부와 테스트 품질 이슈를 심각도별로 요약 보고.
