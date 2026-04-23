---
description: 구현만 수행 (development 에이전트 단독 실행 - 모드 3)
---

파이프라인 전체를 돌리지 않고 **구현만** 수행한다. development 에이전트를 모드 3(단독 실행)으로 호출한다.

**요청 내용:**
$ARGUMENTS

**실행 방식:**

Agent tool로 `development` 에이전트를 호출하며, 모드 3으로 동작하도록 다음을 전달한다:
- 사용자 요청 내용
- 모드 3: 단독 실행이므로 문서도 직접 관리해야 함을 명시

**development 에이전트 모드 3 동작:**
- 구현 전: `docs/exec-plans/YYYY-MM-DD-제목.md` 계획 문서 작성
- 구현 중: TDD 사이클 (Red → Green → Refactor)
- 구현 후: 변경 파일 목록과 구현 요약을 반환 (deliverable.md 생성 **하지 않음**)

**완료 기준:**
- `./gradlew build` 성공
- `./gradlew test` 전체 통과
- exec-plan 문서 작성 완료

**인수인계 문서가 필요하면:**
`/doc-handoff` 커맨드를 추가로 실행하여 `deliverable.md`를 생성한다.
