# fullstack-nextjs-fastapi 명령어 가이드

Next.js + FastAPI 풀스택 프로젝트를 위한 개발 파이프라인 플러그인.
요구사항 분석부터 배포 준비, Notion 인수인계까지 한 흐름으로 처리한다.

## 도메인 치환 안내

이 플러그인의 에이전트/스킬은 **mydata-ai-poc(건강검진 데이터 관리 대시보드)** 의 자산을 기반으로 만들어졌다.
따라서 일부 페르소나/예시에 **"건강검진"**, **"진단 데이터"**, **"LLM 분석"** 같은 도메인 용어가 등장한다.

**사용 시 본인 프로젝트 도메인으로 자연스럽게 치환된다** — Claude가 사용자 프로젝트의 `CLAUDE.md`와 코드베이스를 먼저 읽고 컨텍스트를 잡기 때문에, 도메인 예시는 "이런 스타일로 분석/구현하라"는 **패턴 가이드** 정도로 작동한다.

다만 다음 두 가지는 직접 손대는 게 정확도를 높인다:
1. `CLAUDE.md` 의 `<<placeholder>>` (스택/도메인 설명) — `/init` 으로 생성된 템플릿에서 본인 프로젝트 값으로 교체
2. 도메인 깊은 작업(예: 결제, 인증, 의료 등 규제 영역)이면 `/spec` 단계에서 도메인 컨텍스트를 명시적으로 추가 입력

## 초기 세팅

```bash
/init
```

다음 파일/디렉토리가 생성된다:
- `docs/{design,exec-plans,output,architecture,product-specs}/`
- `CLAUDE.md` (도메인 placeholder 포함)
- `.mcp.json` (MySQL MCP, 환경변수 플레이스홀더)

생성 후:
1. `CLAUDE.md` 의 `<<...>>` 부분을 본인 프로젝트 값으로 교체
2. `.mcp.json` 의 `${MYSQL_HOST}` 등 환경변수를 셸/`.envrc` 에 설정 (또는 직접 값 입력 — 단, **git 커밋 금지**)

## MySQL MCP 설정

이 플러그인의 일부 에이전트(특히 `analyze-requirements`, `development-plan`)는 MySQL 스키마를 직접 조회한다.
`.mcp.json` 템플릿:

```json
{
  "mcpServers": {
    "mysql": {
      "command": "npx",
      "args": ["-y", "@benborla29/mcp-server-mysql"],
      "env": {
        "MYSQL_HOST": "${MYSQL_HOST}",
        "MYSQL_PORT": "${MYSQL_PORT:-3306}",
        "MYSQL_USER": "${MYSQL_USER}",
        "MYSQL_PASS": "${MYSQL_PASS}",
        "MYSQL_DB": "${MYSQL_DB}"
      }
    }
  }
}
```

환경변수는 셸 또는 `.envrc` (direnv)에 정의:
```bash
export MYSQL_HOST=...
export MYSQL_USER=...
export MYSQL_PASS=...
export MYSQL_DB=...
```

MySQL 외 다른 DB를 쓴다면 해당 MCP 서버로 교체하면 된다.

## 파이프라인 아키텍처

```
Stage 1: SPEC (요구사항 분석)
  ↓
Stage 2: PLAN (기술 플랜)
  ↓
Stage 3-4: BUILD-QA LOOP (TDD 구현 + 검증, 최대 3회)
  ├─ 3. BUILD (구현)
  └─ 4. QA (테스트 + 리뷰)
  ↓
Stage 5: DEPLOY (배포 준비)
  ↓
Stage 6: HANDOFF (결과물 통합)
  ↓
Stage 7: NOTION (Notion 업로드)
```

## 사용 가능한 명령어

| 명령어 | 설명 | 대상 |
|--------|------|------|
| `/init` | 프로젝트 docs 구조 + CLAUDE.md + .mcp.json 초기화 | 세팅 |
| `/pipeline <요구사항>` | 전체 파이프라인 자동 실행 (오케스트레이터) | 풀스택 |
| `/spec <요구사항>` | 요구사항 분석 (Figma/Notion/API 명세 → requirements.md) | 풀스택 |
| `/plan <feature-name>` | 기술 플랜 작성 (프론트+백) | 풀스택 |
| `/build <요구사항>` | 구현 (프론트엔드 + 백엔드) | 풀스택 |
| `/test` | 테스트 실행 + 품질 리뷰 | 풀스택 |
| `/review [mode]` | 코드 리뷰 (wip/staged/unpushed) | 풀스택 |
| `/qa` | 전체 검증 (QA 리포트) | 풀스택 |
| `/deploy-check` | 배포 점검 | 풀스택 |
| `/doc-handoff` | 결과물 통합 문서 작성 | 풀스택 |
| `/commands` | 명령어 목록 표시 | 정보 |

## 에이전트 구성

상위 오케스트레이터 1개 + 단계별 전담 에이전트 7개:

- `pipeline` — 7단계 파이프라인 오케스트레이터
- `analyze-requirements` — Figma/Notion/API 명세 → 요구사항 문서
- `development-plan` — 요구사항 → 풀스택 실행 계획
- `development` — 플랜 → Next.js + FastAPI 코드 구현
- `test-qa` — 구현 검증 (pytest + npm run build + 시나리오)
- `deploy-prep` — 배포 전 체크리스트
- `doc-handoff-writer` — 인수인계 문서 생성
- `notion-upload` — Notion 페이지로 업로드

각 에이전트는 단독으로도 호출 가능 (`Agent(<name>)` 또는 슬래시 커맨드).

## 리뷰 모드

`/review` 명령어는 비교 범위를 mode로 받는다:

- **wip** (기본): `git diff HEAD` — 작업 중인 변경사항
- **staged**: `git diff --staged` — 커밋 직전 변경사항
- **unpushed**: `git diff @{u}...HEAD` — PR 올리기 전 (추천)

## 산출물 구조

```
docs/design/{feature-name}/         ← 설계 (입력)
├── requirements.md                  (/spec)
└── plan.md                          (/plan)

docs/exec-plans/                     ← 실행 기록 (단건)
└── YYYY-MM-DD-제목.md               (/build 단독)

docs/output/{feature-name}/          ← 결과물 (출력)
├── test-report.md                   (/test)
├── review-report.md                 (/review)
├── qa-report.md                     (/qa)
├── deploy-checklist.md              (/deploy-check)
└── deliverable.md                   (/doc-handoff)
```

폴더 역할: **design(설계) → exec-plans(실행) → output(결과)**

## 외부 통합

- **Figma MCP** — `analyze-requirements` 가 디자인 컨텍스트/스크린샷 수집
- **Notion MCP** — `analyze-requirements` 가 기획 문서 fetch, `notion-upload` 가 결과물 업로드
- **MySQL MCP** — 스키마 조회 (분석/플랜 단계)

이 MCP들은 Claude Code 사용자 환경에 별도로 인증되어 있어야 한다 (`mcp__claude_ai_Figma__*`, `mcp__claude_ai_Notion__*`).

## 기반 프로젝트

이 플러그인은 [mydata-ai-poc](../../../) 의 `.claude/` 자산을 마켓플레이스 컨벤션에 맞게 재배치한 것이다.
프론트 전용 / 백 전용 분리 버전(`frontend-nextjs`, `backend-fastapi`)은 추후 별도 플러그인으로 분리 예정.
