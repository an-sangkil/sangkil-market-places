# docs/references/

개발 참조 가이드 디렉토리. 에이전트가 코드 작성/리뷰/검증 시 **단일 기준 문서**로 참조한다.

## 파일 구조

```
references/
├── planning/                         ← PLAN 단계에서 참조
│   ├── execution-plan-template.md    ← plan.md 출력 형식
│   └── requirement-analysis-checklist.md  ← 요구사항 분석 체크리스트
├── development/                      ← BUILD 단계에서 참조
│   ├── testing-patterns.md           ← 테스트 작성 원칙과 검증 기준
│   └── security-checklist.md         ← 보안 체크리스트
├── qa/                               ← QA 단계에서 참조
│   └── triage-procedure.md           ← QA 이슈 수정 절차 (Triage & Prove-It)
└── anti-rationalization.md           ← 전 단계 공통: 에이전트 변명 차단 규칙
```

## 작성 규칙

- 이 디렉토리의 파일은 에이전트의 **행동 기준** 역할을 한다.
- 프로젝트 초기에 템플릿을 기반으로 생성하고, 프로젝트 기술 스택에 맞게 커스터마이즈한다.
- 에이전트가 이 문서와 다른 판단을 하려면, 사용자의 명시적 승인이 필요하다.

## 에이전트별 참조 맵

| 에이전트 | 참조 문서 |
|----------|-----------|
| `development-plan` | `planning/execution-plan-template.md`, `planning/requirement-analysis-checklist.md` |
| `development` | `development/testing-patterns.md`, `development/security-checklist.md` |
| `test-qa` | `development/testing-patterns.md`, `qa/triage-procedure.md`, `development/security-checklist.md` |
| 전체 | `anti-rationalization.md` |
