---
description: 테스트 실행 + 테스트 품질 리뷰 (코드 QA 체크리스트는 스킵)
---

현재 변경사항의 테스트를 실행하고 테스트 품질만 리뷰한다. 코드 QA 체크리스트는 수행하지 않는다.

**요청 내용 (선택):**
$ARGUMENTS

**실행 방식:**

1. **변경사항 파악** — Bash로 `git status`, `git diff --name-only HEAD`로 수정/생성된 파일 목록 확인

2. **test-qa 에이전트 호출** — Agent tool로 다음을 전달:
   - **mode: `test`**
   - 변경된 파일 목록
   - 개발 플랜 경로 (있으면)

**test-qa 에이전트 동작 (mode=test):**
- `./gradlew test` 실행하여 증거 확보
- 테스트 품질 리뷰 (커버리지, DAMP, 상태기반, Mock 적절성)
- 코드 QA 체크리스트는 **스킵**
- 산출물: `docs/output/{feature-name}/test-report.md`

**결과 보고:**
테스트 통과 여부와 테스트 품질 이슈를 간단히 요약하여 사용자에게 보고한다.
