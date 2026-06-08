---
description: 테스트 실행 + 품질 리뷰
---

테스트를 실행하고 품질을 리뷰합니다.

1. 백엔드 테스트: `cd mydata-admin-server && .venv/bin/pytest -v`
2. 프론트엔드 타입 체크: `cd mydata-admin-client && npm run type-check`
3. 프론트엔드 빌드 체크: `cd mydata-admin-client && npm run build`
4. 테스트 품질 리뷰 (커버리지, 엣지 케이스)
5. test-report.md를 저장합니다

산출물: `docs/output/{feature-name}/test-report.md`
