---
description: 전체 검증 (테스트 실행 + 테스트 품질 리뷰 + 코드 QA 체크리스트)
---

현재 작업 중인 변경사항에 대해 **전체 QA 검증**을 수행한다.

**요청 내용 (선택):**
$ARGUMENTS

## 실행 방식

### 1) 변경사항 파악
`git status`, `git diff --name-only HEAD`로 수정/생성된 파일 목록 확인.

### 2) 컨텍스트 확인
관련 개발 플랜이 `document/exec-plans/{domain}/plan.md`에 있는지 확인. 없으면 사용자에게 "도메인명" 또는 "검증 기준"을 물어본다.

### 3) test-qa 에이전트 호출

Agent tool로 다음을 전달:
- **focus: `full`** (테스트 실행 + 테스트 품질 리뷰 + 코드 QA 체크리스트 전체)
- 변경된 파일 목록
- 개발 플랜 경로 (있으면)
- 사용자가 추가로 요청한 검증 항목 (있으면)

**참고:** 빠른 검증이 필요하면 `/test` (테스트만) 또는 `/review` (리뷰만)를 사용.

## 에이전트 동작 (focus=full)

- 10년 경력 시니어 QA 엔지니어 관점으로 동작
- 테스트 실행: `./gradlew test` 로 증거 확보
- 테스트 품질 리뷰 (커버리지/DAMP/상태 기반/Mock 적절성)
- 코드 QA 체크리스트 (기능/코드/보안/데이터/계층 아키텍처)
- 이슈는 심각도(Critical/Major/Minor)와 재현 조건 포함

## 산출물

`document/exec-plans/{domain}/qa-report.md`

## 결과 보고

QA 리포트의 "종합 판단"(PASS/FAIL)과 핵심 이슈를 사용자에게 요약 보고.
