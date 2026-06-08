---
description: 전체 개발 파이프라인 자동 실행
---

전체 파이프라인을 실행합니다.

요구사항:
$ARGUMENTS

Agent(analyze-requirements) → requirements.md
→ 사용자 승인
Agent(development-plan) → plan.md
→ 사용자 승인
LOOP (최대 3회):
  Agent(development) → 구현
  Agent(test-qa, mode=full) → qa-report.md
  PASS → 다음 단계
  FAIL → 재시도
Agent(deploy-prep) → deploy-checklist.md
→ 사용자 승인
Agent(doc-handoff-writer) → deliverable.md
