---
description: 검증된 스펙을 기반으로 6단계 실행계획 + TODO 작성 (spec 이후 단계)
---

이미 작성된 스펙/요구사항 문서를 기반으로 **6단계 실행계획과 TODO 리스트**를 작성한다.

**요청 내용 (도메인명 또는 스펙 경로):**
$ARGUMENTS

## 실행 방식

### 1) 스펙 경로 확인

`$ARGUMENTS`로부터 스펙 경로를 결정한다:

- **도메인명이 주어진 경우**: `docs/design/{domain}/requirements.md` 사용
- **파일 경로가 주어진 경우**: 해당 경로 사용
- **인자가 없는 경우**:
  - `ls -lt docs/design/*/requirements.md | head -5`로 최근 스펙 후보 제시
  - 사용자에게 어떤 스펙으로 플랜을 작성할지 확인

스펙 파일이 존재하지 않으면 `/spec`을 먼저 실행하도록 안내하고 중단.

### 2) requirements-todo-planner 에이전트 호출

Agent tool로 `requirements-todo-planner` 에이전트를 호출하며 스펙 경로(또는 intake handoff YAML)를 전달한다.

## 산출물

`docs/design/{domain}/plan.md`

포함 내용:
- 요구사항 구체화 / 범위·가정·리스크
- 6단계 실행계획
- TODO 리스트 (priority/step/agent/task/blocked_by/done_when/effort/parallel/risk)
- 단계별 에이전트 호출 순서
- `backend-implementation` handoff YAML

## 결과 보고

플랜 요약을 사용자에게 보고한 뒤 다음 단계 안내:

```
✅ 실행계획 작성 완료
📄 산출물: docs/design/{domain}/plan.md

➡️ 다음 단계:
   /build {domain}   → 코드 구현 (TDD)
```

## 언제 쓰나

- `/spec`으로 요구사항을 확정한 뒤 플랜만 별도로 작성
- 기존 스펙 문서로 플랜만 다시 작성하고 싶을 때
- 구현 전에 플랜을 리뷰받고 싶을 때
