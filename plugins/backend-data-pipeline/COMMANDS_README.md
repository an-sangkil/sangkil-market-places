# Claude Code Commands — backend-data-pipeline

이벤트 드리븐 데이터 파이프라인 프로젝트를 위한 슬래시 커맨드 목록이다. 각 커맨드는 `commands/<이름>.md` 파일로 정의되어 있으며, Claude Code에서 `/<커맨드명>` 형태로 호출한다.

## 전체 목록

| Command | 설명 | 호출 대상 |
|---|---|---|
| `/pipeline` | 전체 파이프라인 (7단계 자동) | `pipeline` |
| `/spec` | 요구사항/스펙(PRD) 작성만 | `analyze-requirements` |
| `/plan` | 스펙 기반 개발 플랜 작성 | `development-plan` |
| `/build` | 구현만 수행 (TDD) | `development` |
| `/test` | 테스트 실행 + 테스트 품질 리뷰 | `test-qa` (mode=test) |
| `/review` | 코드 리뷰만 (3가지 모드) | `test-qa` (mode=review) |
| `/qa` | 전체 검증 (테스트 + 리뷰 + 체크리스트) | `test-qa` (mode=full) |
| `/deploy-check` | 배포 전 체크리스트 | `deploy-prep` |
| `/doc-handoff` | 인수인계 문서 통합 | `doc-handoff-writer` |
| `/notion-upload` | 결과물 Notion 업로드 | `notion-upload` |
| `/commands` | 이 커맨드 목록 보기 | — |

---

## 개발 라이프사이클

이 플러그인은 **7단계 파이프라인 방식**을 따른다. 각 단계는 **명확한 입력/산출물/게이트**를 가진다.

### 전체 흐름도

```
┌──────────┐  ┌──────────┐  ┌──────────────┐  ┌──────────┐
│  1. SPEC │→ │  2. PLAN │→ │ 3~4. BUILD   │→ │ 5.DEPLOY │
│ 요구사항 │  │ 개발플랜 │  │  ⇄ QA 루프   │  │  준비    │
└──────────┘  └──────────┘  └──────────────┘  └──────────┘
   기획게이트    리뷰게이트     자동루프(3회)     배포게이트
                                                    ↓
                              ┌──────────┐  ┌──────────┐
                              │ 7. SHARE │← │ 6.HANDOFF│
                              │ 노션공유 │  │ 문서통합 │
                              └──────────┘  └──────────┘
                                 공유게이트    문서게이트
```

### 각 단계 정의

| # | 단계 | 목적 | 입력 | 산출물 | 게이트 통과 조건 | 커맨드 |
|---|---|---|---|---|---|---|
| 1 | **SPEC** | 무엇을 만들지 정의 | 요구사항/기획/정책 | `requirements.md` | 목표/범위/AC 명확 | `/spec` |
| 2 | **PLAN** | 어떻게 만들지 설계 | 확정된 스펙 | `plan.md` | 태스크+의존성+리스크 | `/plan` |
| 3~4 | **BUILD⇄QA** | 구현 + 자동 검증 | 플랜 | 소스 코드 + QA 리포트 | QA PASS (최대 3회) | `/build` + `/qa` |
| 5 | **DEPLOY** | 배포 준비 | 변경 전체 | `deploy-checklist.md` | 체크리스트 통과 | `/deploy-check` |
| 6 | **HANDOFF** | 산출물 통합 | 전체 산출물 | `deliverable.md` | 섹션 완비 | `/doc-handoff` |
| 7 | **SHARE** | 노션 공유 | deliverable.md | Notion 페이지 URL | 업로드 완료 | `/notion-upload` |

### QA 자동 루프 (3~4단계)

파이프라인 모드에서 BUILD와 QA는 자동으로 반복된다:

```
qa_attempt = 0
LOOP (최대 3회):
  qa_attempt += 1
  BUILD → development 에이전트 (TDD: Red → Green → Refactor)
  QA    → test-qa 에이전트 (테스트 + 리뷰 + 체크리스트)

  PASS → 5단계로 진행
  FAIL + 새 이슈 → 자동 재시도
  FAIL + 동일 이슈 반복 → 중단, 수동 개입
  FAIL + 3회 초과 → 중단
```

### 게이트 의미

- **기획 게이트 (1→2)**: 스펙이 모호하면 플랜이 의미 없다 — 목표/범위/AC가 명확해야 통과
- **리뷰 게이트 (2→3)**: 플랜의 리스크가 수용 가능해야 통과
- **QA 게이트 (4→5)**: QA PASS 판정 (자동 루프)
- **배포 게이트 (5→6)**: 배포 체크리스트 항목 확인
- **문서 게이트 (6→7)**: 인수인계 문서 섹션 완비
- **공유 게이트 (7)**: Notion 업로드 성공

### 실행 모드

**자동 모드 — 파이프라인:**

```
/pipeline <요구사항>    → 7단계 자동 (단계별 승인 포함)
```

**수동 모드 — 단계별:**

```
/spec <요구사항>         → 스펙 작성
/plan <feature-name>     → 플랜 작성
/build <요구사항>        → 구현
/review unpushed         → 코드 리뷰
/test                    → 테스트
/qa                      → 전체 검증
/deploy-check            → 배포 체크리스트
/doc-handoff             → 인수인계 문서
/notion-upload           → Notion 업로드
```

### 문서 작성 시점

| 문서 종류 | 경로 | 작성 시점 |
|---|---|---|
| 요구사항 | `docs/design/{feature-name}/requirements.md` | SPEC 단계 |
| 개발 플랜 | `docs/design/{feature-name}/plan.md` | PLAN 단계 |
| 테스트 리포트 | `docs/output/{feature-name}/test-report.md` | TEST 단계 |
| 리뷰 리포트 | `docs/output/{feature-name}/review-report.md` | REVIEW 단계 |
| QA 리포트 | `docs/output/{feature-name}/qa-report.md` | QA 단계 |
| 배포 체크리스트 | `docs/output/{feature-name}/deploy-checklist.md` | DEPLOY 단계 |
| 인수인계 문서 | `docs/output/{feature-name}/deliverable.md` | HANDOFF 단계 |

### 단계 스킵 허용 여부

| 스킵 | 언제 | 리스크 |
|---|---|---|
| SPEC 스킵 | 요구사항이 이미 명확한 간단한 변경 | 낮음 |
| PLAN 스킵 | 1~2시간 내 끝나는 버그픽스 | 낮음 |
| REVIEW 스킵 | — | **금지** — PR 전 필수 |
| TEST 스킵 | — | **금지** — 증거 없는 배포는 없음 |
| QA 스킵 | 설정 파일 변경 등 비-코드 변경 | 중간 |
| DEPLOY 스킵 | 개발 환경 변경만 | 낮음 |
| HANDOFF 스킵 | 사내 공유 불필요한 경우 | 낮음 |

---

## 각 커맨드별 세부

### `/pipeline <요구사항>`
- **전체 파이프라인** (7단계 자동)
- `pipeline` 에이전트 호출
- 1/2/5/6/7단계는 사용자 승인, 3/4단계는 자동 QA 루프

### `/spec <요구사항>`
- **요구사항/스펙(PRD) 작성만** 수행
- `analyze-requirements` 에이전트 호출
- 산출물: `docs/design/{feature-name}/requirements.md`
- 다음 단계: `/plan`

### `/plan <domain 또는 스펙 경로>`
- 이미 작성된 스펙을 기반으로 **개발 플랜** 작성
- `development-plan` 에이전트 호출
- 산출물: `docs/design/{feature-name}/plan.md`
- 다음 단계: `/build`

### `/build <domain 또는 요구사항>`
- **구현만 단독 실행**
- `development` 에이전트 호출 (모드 3: 단독)
- 구현 순서: Entity → Mapper → Parser → Saver → Collector → Job → Config
- 완료 기준: 빌드 성공 + 기존 테스트 미파손

### `/test`
- `./gradlew test` 실행 + 테스트 품질 리뷰
- 코드 QA 체크리스트는 **스킵** (빠름)
- `test-qa` 에이전트 (mode=test)
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

### `/deploy-check`
- 배포 전 체크리스트 작성
- `deploy-prep` 에이전트 호출
- DB 마이그레이션, 환경변수, 롤백 절차 검증
- 산출물: `deploy-checklist.md`

### `/doc-handoff [feature-name]`
- 모든 산출물을 하나의 인수인계 문서로 통합
- `doc-handoff-writer` 에이전트 호출
- 산출물: `deliverable.md`

### `/notion-upload [경로 또는 도메인]`
- `deliverable.md`를 Notion 페이지로 업로드
- 중복 페이지 확인 후 create/update 선택

---

## 참고

- 에이전트 정의: `agents/`
- 스킬 정의: `skills/`
- 프로젝트별 아키텍처 규칙: 적용 대상 프로젝트의 `CLAUDE.md` 및 `doc/` 참조
