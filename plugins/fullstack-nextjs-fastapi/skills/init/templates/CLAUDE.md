# <<프로젝트명>>

<<도메인 한 줄 설명>> (예: "건강검진 데이터 기반 AI 분석 및 통계 관리 대시보드")

## 언어 설정

- 모든 응답, 주석, 커밋 메시지, 문서는 한국어로 작성한다
- 기술 용어는 필요시 영어 병기 (예: 벡터 스토어(Vector Store))

## 프로젝트 구조

```
<<프로젝트명>>/
├── <<client-dir>>/     # Next.js 프론트엔드 (예: admin-client)
├── <<server-dir>>/     # FastAPI 백엔드 (예: admin-server)
├── docs/
│   ├── design/              # 설계 문서 (/spec, /plan 산출물)
│   ├── exec-plans/          # 실행 기록 (/build 단독 실행 계획)
│   ├── output/              # 결과물 (/review, /qa, /deploy-check, /doc-handoff)
│   ├── product-specs/       # 제품 참고 문서 (읽기 전용)
│   └── architecture/        # 아키텍처 문서
├── .mcp.json                # MCP 서버 설정 (MySQL 등)
└── CLAUDE.md
```

## 문서 구조

```
docs/
├── design/                          ← 설계 (왜, 어떻게)
│   └── {feature}/
│       ├── requirements.md          ← /spec
│       └── plan.md                  ← /plan
│
├── exec-plans/                      ← 실행 기록 (단건)
│   └── YYYY-MM-DD-제목.md           ← /build (단독)
│
├── output/                          ← 결과물 (검증, 배포, 인수인계)
│   └── {feature}/
│       ├── test-report.md           ← /test
│       ├── review-report.md         ← /review
│       ├── qa-report.md             ← /qa
│       ├── deploy-checklist.md      ← /deploy-check
│       └── deliverable.md           ← /doc-handoff
│
├── product-specs/                   ← 참고 (읽기 전용)
└── architecture/                    ← 아키텍처
```

## 기술 스택

### Frontend (<<client-dir>>)
- **프레임워크**: Next.js 16 (Turbopack)
- **언어**: TypeScript (strict)
- **UI**: React 19 + Tailwind CSS v4
- **데이터 패칭**: TanStack React Query + Axios
- **차트**: Recharts (선택)
- **컴포넌트**: Radix UI, Lucide React (선택)

### Backend (<<server-dir>>)
- **프레임워크**: FastAPI
- **언어**: Python 3.12
- **ORM**: SQLAlchemy 2.0 (async)
- **검증**: Pydantic v2
- **LLM** (해당 시): LangChain + Google Gemini / OpenAI / Anthropic
- **DB 드라이버**: asyncmy (MySQL async)
- **서버**: uvicorn

### Database
- MySQL 8.0 (개발 서버 정보는 본인 프로젝트에 맞게 작성)
- 주요 테이블: <<도메인 테이블 예시>> (예: diagnostic_report, diagnostic_observation)

## 실행 방법

```bash
# 백엔드 실행
cd <<server-dir>>
source .venv/bin/activate  # 또는 .venv/bin/python3 직접 사용
uvicorn src.main:app --reload --port 8000

# 프론트엔드 실행
cd <<client-dir>>
npm run dev
```

## 주요 API (예시 — 본인 프로젝트에 맞게 교체)

- `GET /api/health` — 서버 상태 확인
- `POST /api/v1/<<endpoint>>` — <<설명>>
- `GET /api/v1/<<resource>>` — <<설명>>

## 프론트엔드 라우팅 (예시)

- `/` — <<메인 페이지 설명>>
- `/<<route1>>` — <<설명>>
- `/<<route2>>` — <<설명>>

## 테스트

```bash
# 백엔드
cd <<server-dir>> && .venv/bin/pytest -v

# 프론트엔드 타입 체크
cd <<client-dir>> && npm run type-check

# 프론트엔드 빌드 체크
cd <<client-dir>> && npm run build
```

## Git 워크플로우

- **브랜치**: feature/*, bugfix/*, refactor/*
- **커밋 메시지**: Conventional Commits 형식 (설명은 한국어)
  - `feat: 설명` — 새로운 기능
  - `fix: 설명` — 버그 수정
  - `refactor: 설명` — 리팩토링
  - `docs: 설명` — 문서 수정
  - `chore: 설명` — 빌드/설정 변경
  - `test: 설명` — 테스트 추가/수정
  - `style: 설명` — 코드 포맷 변경
  - `perf: 설명` — 성능 개선
  - 스코프 예시: `feat(client): 대시보드 차트 추가`, `fix(server): 인증 토큰 만료 처리`
  - Breaking change: `feat!: 설명` 또는 body에 `BREAKING CHANGE:` 명시

## 주의사항

- `.env` 파일에 API 키 저장 (절대 커밋 금지)
- `.mcp.json` 의 DB 접속 정보는 환경변수(`${MYSQL_HOST}` 등)로 분리 (평문 커밋 금지)
- `docs/product-specs/`는 참고 문서이므로 구현 중 수정하지 않음
