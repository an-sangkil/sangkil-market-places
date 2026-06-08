---
description: 결과물 통합 문서 작성
---

파이프라인 산출물을 하나의 인수인계 문서로 통합합니다.

대상: $ARGUMENTS

수집 대상:
- docs/design/{feature-name}/requirements.md
- docs/design/{feature-name}/plan.md
- docs/output/{feature-name}/qa-report*.md
- docs/output/{feature-name}/deploy-checklist.md
- 변경 파일 목록 (git diff)

산출물 구조:
1. 개요 (목표, 범위)
2. 구현 상세 (변경 파일, 핵심 구현)
3. API 명세 (해당 시)
4. 데이터 모델 변경 (해당 시)
5. QA 결과 요약
6. 배포 가이드

산출물: `docs/output/{feature-name}/deliverable.md`
