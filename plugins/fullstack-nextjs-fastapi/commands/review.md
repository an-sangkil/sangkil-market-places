---
description: 코드 리뷰 (테스트 실행 없음)
---

코드를 리뷰합니다. 테스트는 실행하지 않습니다.

모드: $ARGUMENTS (wip/staged/unpushed, 기본: wip)

리뷰 모드:
- wip: `git diff HEAD`
- staged: `git diff --staged`
- unpushed: `git diff @{u}...HEAD`

체크리스트:
- 기능: 요구사항 구현 여부, 예외 처리
- 코드 품질: 패턴 일관성, 네이밍, 불필요 코드
- 보안: SQL 인젝션, XSS, 입력 검증, 환경변수 노출
- 프론트엔드: TypeScript 타입 안전성, 접근성, 반응형
- 백엔드: async/await 일관성, 에러 핸들링, Pydantic 검증

산출물: `docs/output/{feature-name}/review-report.md`
