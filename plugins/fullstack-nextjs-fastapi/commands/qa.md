---
description: 전체 검증 (테스트 + 리뷰)
---

테스트 실행과 코드 리뷰를 모두 수행합니다.

1. 백엔드 테스트: `cd mydata-admin-server && .venv/bin/pytest -v`
2. 프론트엔드 타입 체크 + 빌드 체크
3. 코드 QA 체크리스트 (보안, 품질, 기능)
4. 이슈 목록 및 심각도 평가
5. 최종 판정: PASS / FAIL

산출물: `docs/output/{feature-name}/qa-report.md`
