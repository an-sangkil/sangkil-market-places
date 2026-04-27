# Claude Code Commands — backend-data-pipeline

이벤트 드리븐 데이터 파이프라인 프로젝트를 위한 슬래시 커맨드 목록이다. 각 커맨드는 `commands/<이름>.md` 파일로 정의되어 있으며, Claude Code에서 `/<커맨드명>` 형태로 호출한다.

## 전체 목록

| Command | 설명 | 호출 대상 |
|---|---|---|
| `/pipeline` | 전체 파이프라인 실행 (9단계 자동) | `pipeline` |
| `/spec` | 요구사항/스펙(PRD) 작성만 | `analyze-requirements` |
| `/plan` | 스펙 기반 개발 플랜 작성 | `development-plan` |
| `/build` | 구현만 수행 (TDD, 단독 실행 모드 3) | `development` |
| `/test` | 테스트 실행 + 테스트 품질 리뷰 | `test-qa` (mode=test) |
| `/review` | 코드 리뷰만 (빠름, 3가지 모드) | `test-qa` (mode=review) |
| `/qa` | 전체 검증 (테스트 + 리뷰 + 체크리스트) | `test-qa` (mode=full) |
| `/deploy-check` | 배포 전 체크리스트 | `deploy-prep` |
| `/doc-handoff` | 인수인계 문서 통합 | `doc-handoff-writer` |
| `/notion-upload` | 결과물 Notion 업로드 | `notion-upload` |
| `/commands` | 이 커맨드 목록 보기 | — |

---

## 개발 라이프사이클

이 플러그인은 **spec-driven + TDD** 방식의 **9단계 파이프라인**을 따른다. 각 단계는 **명확한 입력/산출물/게이트**를 가진다.

### 전체 흐름도

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  1. SPEC │→ │  2. PLAN │→ │ 3. BUILD │→ │ 4. REVIEW│→ │  5. TEST │
│ 요구사항  │  │  기술플랜 │  │   구현   │  │ 코드점검 │  │테스트실행│
└──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
   기획게이트    리뷰게이트    TDD자동    PR직전게이트  테스트게이트
                                                           ↓
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ 9.SHARE  │← │8.HANDOFF │← │ 7.DEPLOY │← │  6. QA   │
│ 노션공유 │  │ 문서통합 │  │ 배포준비 │  │ 전체검증 │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
   공유게이트   인수게이트   배포게이트    최종게이트
```

### 각 단계 정의

| # | 단계 | 목적 | 입력 | 산출물 | 게이트 통과 조건 | 커맨드 |
|---|---|---|---|---|---|---|
| 1 | **SPEC** | 무엇을 만들지 정의 | 사용자 요구사항, API 명세, Figma | `requirements.md` | 목표/경계/AC 명확 | `/spec` |
| 2 | **PLAN** | 어떻게 만들지 설계 | 확정된 스펙 | `plan.md` | 태스크 분해 + 리스크 식별 | `/plan` |
| 3 | **BUILD** | TDD로 구현 | 플랜 | 소스 코드 + 테스트 | `./gradlew build` 성공 | `/build` |
| 4 | **REVIEW** | 코드 품질 점검 | 변경 코드 | `review-report.md` | Critical 이슈 0 | `/review` |
| 5 | **TEST** | 테스트 통과 확인 | 구현+테스트 코드 | `test-report.md` | 전체 통과 | `/test` |
| 6 | **QA** | 기능/보안/데이터 전수검증 | 변경 전체 | `qa-report.md` | PASS 판정 | `/qa` |
| 7 | **DEPLOY** | 배포 준비 | QA PASS 결과 | `deploy-checklist.md` | 체크리스트 완료 | `/deploy-check` |
| 8 | **HANDOFF** | 산출물 통합 (인수인계 문서) | 1~7 단계 산출물 + 변경 코드 | `deliverable.md` | 6개 섹션 모두 채움 | `/doc-handoff` |
| 9 | **SHARE** | Notion 공유 | `deliverable.md` | Notion 페이지 URL | 업로드 완료 | `/notion-upload` |

### 게이트 의미

각 단계 사이의 **게이트**는 다음 단계로 넘어가기 전 통과해야 하는 조건이다.

- **기획 게이트 (1→2)**: 스펙이 모호하면 플랜이 의미 없다 — 목표/경계/AC가 명확해야 통과
- **리뷰 게이트 (2→3)**: 플랜의 리스크가 수용 가능해야 통과
- **TDD 자동 (3 내부)**: Red → Green → Refactor 사이클 자동
- **PR 직전 게이트 (3→4)**: 코드 리뷰로 Critical 이슈 제거
- **테스트 게이트 (4→5)**: 모든 테스트 통과
- **최종 게이트 (5→6)**: QA 체크리스트 전수 통과 (기능/보안/데이터/파싱)
- **배포 게이트 (6→7)**: 배포 체크리스트 (DB 마이그레이션, 환경변수, 롤백 절차)
- **인수 게이트 (7→8)**: 흩어진 산출물을 한 문서(`deliverable.md`)로 통합
- **공유 게이트 (8→9)**: 통합 문서가 외부 독자에게 이해 가능한 상태

### 실행 모드 2가지

**수동 모드 — 단계별 승인:**

각 단계를 수동으로 실행하며 게이트마다 검토.

```
/spec <요구사항>       → 기획/PM 리뷰 후
/plan <feature-name>   → 플랜 리뷰 후
/build <feature-name>  → 구현 (exec-plan 작성 → TDD)
/review unpushed       → 코드 리뷰
/test                  → 테스트
/qa                    → 전체 검증
/deploy-check          → 배포 준비
/doc-handoff           → 산출물 통합 (deliverable.md)
/notion-upload         → 공유
```

**자동 모드 — 파이프라인:**

```
/pipeline <요구사항>   → 1~9단계 자동 진행 (게이트마다 승인 확인)
```

- 1/2/7/8/9단계: 사용자 승인 필요
- 3~6단계: 구현↔QA 자동 루프 (최대 3회)

### 문서 작성 시점

| 문서 종류 | 경로 | 작성 시점 |
|---|---|---|
| 요구사항 | `docs/design/{feature}/requirements.md` | SPEC 단계 |
| 플랜 | `docs/design/{feature}/plan.md` | PLAN 단계 |
| 실행 계획 (단독 BUILD 시) | `docs/exec-plans/YYYY-MM-DD-제목.md` | BUILD 전 |
| 테스트 리포트 | `docs/output/{feature}/test-report.md` | TEST 단계 |
| 리뷰 리포트 | `docs/output/{feature}/review-report.md` | REVIEW 단계 |
| QA 리포트 | `docs/output/{feature}/qa-report.md` | QA 단계 |
| 배포 체크리스트 | `docs/output/{feature}/deploy-checklist.md` | DEPLOY 단계 |
| 결과물 (deliverable) | `docs/output/{feature}/deliverable.md` | HANDOFF 단계 |

### 단계 스킵 허용 여부

| 스킵 | 언제 | 리스크 |
|---|---|---|
| SPEC 스킵 | 요구사항이 이미 명확한 간단한 변경 | 낮음 (작은 변경만) |
| PLAN 스킵 | 1~2시간 내 끝나는 버그픽스 | 낮음 |
| REVIEW 스킵 | — | **금지** — PR 전 필수 |
| TEST 스킵 | — | **금지** — 증거 없는 배포는 없음 |
| QA 스킵 | 설정 파일 변경 등 비-코드 변경 | 중간 |
| DEPLOY 스킵 | 로컬 개발만 | 없음 (배포 안 하니까) |

---

## 각 커맨드별 세부

### `/pipeline <요구사항>`
- 요구사항 분석부터 노션 업로드까지 **전체 자동화**
- 1/2/7/8/9단계는 사용자 승인 게이트
- 3~6단계는 구현↔QA 자동 루프 (최대 3회)

### `/spec <요구사항>`
- **요구사항/스펙(PRD) 작성만** 수행
- 기획 단계에서 스펙 확정 → 리뷰 → 승인 게이트
- 산출물: `docs/design/{feature-name}/requirements.md`
- 다음 단계: `/plan`

### `/plan <domain 또는 스펙 경로>`
- 이미 작성된 스펙을 기반으로 **기술 개발 플랜만** 작성
- 태스크 분해, 파일 변경 계획, 데이터 모델, 검증 계획 포함
- 산출물: `docs/design/{feature-name}/plan.md`
- 다음 단계: `/build`

### `/build <domain 또는 요구사항>`
- 파이프라인 없이 **구현만 단독 실행**
- TDD 사이클 (Red → Green → Refactor) + Anti-Rationalization 준수
- 구현 전 `docs/exec-plans/YYYY-MM-DD-제목.md` 작성
- 구현 후 변경 파일 목록 반환 (deliverable.md는 `/doc-handoff`로 별도 생성)

### `/test`
- `./gradlew test` 실행 + 테스트 품질 리뷰
- 테스트 품질 리뷰 (DAMP, 상태기반, Mock 적절성)
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
- 파이프라인 루프에서 자동 호출됨
- 단독으로 쓸 때: 배포 직전 최종 검증 용도

### `/deploy-check [feature-name]`
- 배포 전 체크리스트 작성
- `deploy-prep` 에이전트 호출
- DB 마이그레이션, 환경변수, 롤백 절차 검증
- 산출물: `docs/output/{feature-name}/deploy-checklist.md`

### `/doc-handoff [feature-name]`
- 1~7단계 산출물들을 **하나의 인수인계 문서**로 통합
- 6개 섹션 (개요/구현/API/데이터모델/QA결과/배포가이드)
- 산출물: `docs/output/{feature-name}/deliverable.md`
- `/build` 단독 사용 후 필요 / `/pipeline` 실행 시 자동 호출

### `/notion-upload [경로 또는 도메인]`
- `deliverable.md` 파일을 Notion 페이지로 업로드
- 경로 미지정 시 최근 수정된 deliverable.md 자동 탐색
- Notion 위치는 사용자가 URL 또는 검색 키워드로 지정

---

## 참고

- 에이전트 정의: `agents/`
- 스킬 정의: `skills/`
- 프로젝트별 아키텍처 규칙: 적용 대상 프로젝트의 `CLAUDE.md` 및 `docs/` 참조
