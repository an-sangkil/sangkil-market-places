---
description: 배포 전 체크리스트 작성 및 배포 준비 상태 검증 (deploy-prep 에이전트)
---

배포 전 체크리스트를 작성하고 배포 준비 상태를 검증한다.

**요청 내용 (선택):**
$ARGUMENTS

**실행 방식:**

1. **대상 파악** — 어떤 기능/변경사항에 대한 배포인지 확인한다:
   - 사용자가 feature-name을 명시했으면 그대로 사용
   - 명시 안 했으면 `git log --oneline -20`으로 최근 커밋을 확인하고 사용자에게 feature-name 확인

2. **변경 사항 수집** — 다음을 정리하여 deploy-prep 에이전트에게 전달:
   - 변경된 파일 목록 (`git diff --name-only main...HEAD`)
   - 개발 플랜/결과물 문서 경로 (`doc/output/{feature-name}/`)
   - DB 마이그레이션 여부
   - 환경변수/설정 변경 여부

3. **deploy-prep 에이전트 호출** — Agent tool로 호출

**산출물:**
`doc/output/{feature-name}/deploy-checklist.md`

**결과 보고:**
체크리스트의 주요 항목(DB 마이그레이션, 환경변수, 롤백 절차 등)과 위험 요소를 사용자에게 요약 보고한다.
