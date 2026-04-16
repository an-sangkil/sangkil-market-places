# Claude Code Commands

이 프로젝트에서 사용하는 슬래시 커맨드 목록이다. 각 커맨드는 `.claude/commands/<이름>.md` 파일로 정의되어 있으며, Claude Code에서 `/<커맨드명>` 형태로 호출한다.

## 전체 목록

| Command | 설명 | 호출 대상 |
|---|---|---|
| `/spec` | 요구사항/스펙(PRD) 작성만 | `requirements-intake-planner` |
| `/plan` | 스펙 기반 6단계 실행계획 + TODO | `requirements-todo-planner` |
| `/build` | 구현만 수행 | `backend-implementation` |
| `/test` | 테스트 실행 + 테스트 품질 리뷰 | `test-qa` (focus=test-only) |
| `/review` | 코드 리뷰만 (3가지 모드) | `test-qa` (focus=review-only) |
| `/qa` | 전체 검증 (테스트 + 리뷰 + 체크리스트) | `test-qa` (focus=full) |
| `/notion-upload` | API 명세를 Notion에 업로드 | `notion-api-spec-upload` |
| `/commands` | 이 커맨드 목록 보기 | — |

**별도 스킬(슬래시 커맨드 아님, 필요 시 직접 호출):**
- `/github-pr-korean` — 한글 PR 생성 표준 절차
- `/backend-predeploy-check` — 배포 전 검증 체크리스트
- `/backend-requirements-planning` — 요구사항 구체화 스킬
- `/setup-platform-sdk` — 의정원 Platform SDK slf4j 제거

---

## 개발 라이프사이클

이 프로젝트는 **spec-driven** 방식을 따른다. 각 단계는 **명확한 입력/산출물/게이트**를 가진다.

### 전체 흐름도

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  1. SPEC │→ │  2. PLAN │→ │ 3. BUILD │→ │ 4. REVIEW│
│ 요구사항 │  │ 실행계획 │  │   구현   │  │ 코드점검 │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
   기획게이트    리뷰게이트      계층준수     PR직전게이트
                                  ↓
              ┌──────────┐  ┌──────────┐  ┌──────────┐
              │ 7.SHARE  │← │  6. QA   │← │  5. TEST │
              │ 노션공유 │  │ 전체검증 │  │ 테스트   │
              └──────────┘  └──────────┘  └──────────┘
                 공유게이트    최종게이트    테스트게이트
```

### 각 단계 정의

| # | 단계 | 목적 | 입력 | 산출물 | 게이트 통과 조건 | 커맨드 |
|---|---|---|---|---|---|---|
| 1 | **SPEC** | 무엇을 만들지 정의 | 요구사항/기획/정책/OCR | `requirements.md` | 목표/범위/AC 명확 | `/spec` |
| 2 | **PLAN** | 어떻게 만들지 설계 | 확정된 스펙 | `plan.md` | 6단계 실행계획 + TODO | `/plan` |
| 3 | **BUILD** | 구현 | 플랜 | 소스 코드 + 테스트 | 컴파일 성공 + 계층 준수 | `/build` |
| 4 | **REVIEW** | 코드 품질 점검 | 변경 코드 | `review-report.md` | Critical 이슈 0 | `/review` |
| 5 | **TEST** | 테스트 통과 확인 | 구현+테스트 | `test-report.md` | 전체 통과 | `/test` |
| 6 | **QA** | 기능/보안/데이터 전수검증 | 변경 전체 | `qa-report.md` | PASS 판정 | `/qa` |
| 7 | **SHARE** | API 명세 공유 | api-spec 파일 | Notion 페이지 URL | 업로드 완료 | `/notion-upload` |

### 게이트 의미

- **기획 게이트 (1→2)**: 스펙이 모호하면 플랜이 의미 없다 — 목표/범위/AC가 명확해야 통과
- **리뷰 게이트 (2→3)**: 플랜의 리스크가 수용 가능해야 통과
- **PR 직전 게이트 (3→4)**: 코드 리뷰로 Critical 이슈 제거
- **테스트 게이트 (4→5)**: 모든 테스트 통과
- **최종 게이트 (5→6)**: QA 체크리스트 전수 통과
- **공유 게이트 (6→7)**: API 명세가 Notion DB와 동기화

### 실행 모드

**수동 모드 — 단계별 승인:**

```
/spec <요구사항>       → 기획/PM 리뷰 후
/plan <domain>         → 플랜 리뷰 후
/build <domain>        → 구현
/review unpushed       → 코드 리뷰
/test                  → 테스트
/qa                    → 전체 검증
/notion-upload <경로>  → API 명세 공유
```

### 문서 작성 시점

| 문서 종류 | 경로 | 작성 시점 |
|---|---|---|
| 요구사항 | `document/exec-plans/{domain}/requirements.md` | SPEC 단계 |
| 실행계획 | `document/exec-plans/{domain}/plan.md` | PLAN 단계 |
| 테스트 리포트 | `document/exec-plans/{domain}/test-report.md` | TEST 단계 |
| 리뷰 리포트 | `document/exec-plans/{domain}/review-report.md` | REVIEW 단계 |
| QA 리포트 | `document/exec-plans/{domain}/qa-report.md` | QA 단계 |
| API 명세 | `document/api-spec/{domain}/*.md` | BUILD 이후 |
| 제품 스펙 | `document/product-specs/{feature}/*.md` | 수시 |

### 단계 스킵 허용 여부

| 스킵 | 언제 | 리스크 |
|---|---|---|
| SPEC 스킵 | 요구사항이 이미 명확한 간단한 변경 | 낮음 |
| PLAN 스킵 | 1~2시간 내 끝나는 버그픽스 | 낮음 |
| REVIEW 스킵 | — | **금지** — PR 전 필수 |
| TEST 스킵 | — | **금지** — 증거 없는 배포는 없음 |
| QA 스킵 | 설정 파일 변경 등 비-코드 변경 | 중간 |

---

## 각 커맨드별 세부

### `/spec <요구사항>`
- **요구사항/스펙(PRD) 작성만** 수행
- `requirements-intake-planner` 에이전트 호출
- 산출물: `document/exec-plans/{domain}/requirements.md`
- 다음 단계: `/plan`

### `/plan <domain 또는 스펙 경로>`
- 이미 작성된 스펙을 기반으로 **6단계 실행계획과 TODO**만 작성
- `requirements-todo-planner` 에이전트 호출
- 산출물: `document/exec-plans/{domain}/plan.md`
- 다음 단계: `/build`

### `/build <domain 또는 요구사항>`
- **구현만 단독 실행**
- `backend-implementation` 에이전트 호출
- 구현 순서: Entity → Repository → Service → Controller
- 완료 기준: 컴파일 성공 + 기존 테스트 미파손

### `/test`
- `./gradlew test` 실행 + 테스트 품질 리뷰
- 코드 QA 체크리스트는 **스킵** (빠름)
- `test-qa` 에이전트 (focus=test-only)
- 산출물: `test-report.md`

### `/review [모드] [추가요청]`
- 코드 리뷰만 (테스트 실행 **스킵**, 가장 빠름)
- 모드 3가지:
  | 모드 | git 기준 | 언제 |
  |---|---|---|
  | (없음) / `wip` | `git diff HEAD` | 작업 중 점검 |
  | `staged` | `git diff --staged` | 커밋 직전 |
  | `unpushed` | `git diff @{u}...HEAD` | **PR 직전 (추천)** |
- 예:
  ```
  /review
  /review staged
  /review unpushed
  /review unpushed 트랜잭션 경계 꼼꼼히 봐줘
  ```

### `/qa`
- **전체 검증** (테스트 실행 + 테스트 품질 + 코드 QA 체크리스트)
- 느리지만 완전함
- 단독으로 쓸 때: 배포 직전 최종 검증 용도

### `/notion-upload [경로 또는 도메인]`
- `document/api-spec/` 하위 마크다운을 Notion API 문서 DB에 업로드
- 중복 페이지 확인 후 create/update 선택
- 경로 미지정 시 최근 수정된 API 명세 후보 제시

---

## 커맨드 추가/수정 방법

새 커맨드를 추가하려면:

1. `.claude/commands/<이름>.md` 파일 생성
2. frontmatter에 `description` 작성 (슬래시 메뉴에 표시됨)
3. 본문에 실행 절차 기술
4. 이 README 표에 추가

```markdown
---
description: 커맨드 설명 (한 줄)
---

실행 절차 본문...

**요청 내용:**
$ARGUMENTS
```

`$ARGUMENTS`는 사용자가 `/<커맨드> <내용>` 형태로 전달한 내용으로 치환된다.

---

## 참고

- 에이전트 정의: `.claude/agents/`
- 스킬 정의: `.claude/skills/`
- 아키텍처 규칙: `document/architecture/backend-application-architecture.md`
- 프로젝트 규칙: `CLAUDE.md`
