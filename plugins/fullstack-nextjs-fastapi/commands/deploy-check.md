---
description: 배포 점검
---

배포 준비 상태를 점검합니다.

대상: $ARGUMENTS

1. 변경 영향 분석 (git diff)
2. 백엔드 빌드 확인 (import 체크, uvicorn 기동 테스트)
3. 프론트엔드 빌드 확인 (npm run build)
4. DB 변경사항 확인 (마이그레이션, DDL)
5. 환경 변수 확인
6. 롤백 계획 작성

산출물: `docs/output/{feature-name}/deploy-checklist.md`
