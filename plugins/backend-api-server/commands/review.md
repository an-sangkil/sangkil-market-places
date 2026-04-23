---
description: 코드 QA 체크리스트만 수행 (테스트 실행 스킵, 빠른 리뷰). 기본/스테이징/미푸시 3가지 모드 지원
---

현재 변경사항에 대해 코드 리뷰만 수행한다. 테스트는 실행하지 않는다.

**요청 내용 (선택):**
$ARGUMENTS

## 리뷰 범위 모드

첫 번째 인자로 모드를 지정할 수 있다. 없으면 **기본(wip)**.

| 모드 인자 | 리뷰 범위 | 사용하는 git 명령 | 언제 쓰나 |
|---|---|---|---|
| (없음) / `wip` | 워킹 트리 변경 | `git diff HEAD` | 작업 중 잠깐 점검 |
| `staged` | 스테이징된 변경만 | `git diff --staged` | 커밋 직전 점검 |
| `unpushed` | 푸시 안 한 커밋 전체 | `git diff @{u}...HEAD` | 푸시/PR 직전 점검 |

## 실행 방식

### 1) 모드 판별 및 안내

`$ARGUMENTS`의 첫 단어를 확인:
- `staged` → staged 모드
- `unpushed` → unpushed 모드
- 그 외 또는 없음 → wip 모드 (기본)

```
🔍 리뷰 모드: {wip|staged|unpushed}
   다른 모드: /review staged  또는  /review unpushed
```

### 2) 변경사항 파악

**wip 모드 (기본):**
```bash
git status
git diff --name-only HEAD
```

**staged 모드:**
```bash
git diff --staged --name-only
git diff --staged --stat
```
스테이징된 파일이 없으면 "스테이징된 변경이 없습니다" 안내 후 종료.

**unpushed 모드:**
```bash
git fetch origin --quiet
git log @{u}..HEAD --oneline
git diff @{u}...HEAD --name-only
```
업스트림 대비 커밋이 없으면 안내 후 종료.
업스트림이 없는 경우 `origin/main...HEAD` 범위로 대체.

### 3) test-qa 에이전트 호출

Agent tool로 다음을 전달:
- **focus: `review-only`** — 코드 QA 체크리스트만 수행
- 리뷰 범위 모드 (wip/staged/unpushed)
- 변경된 파일 목록
- 개발 플랜 경로 (있으면)

## 에이전트 동작 (focus=review-only)

- 테스트 실행 **스킵** (빠름)
- 테스트 품질 리뷰 **스킵**
- 코드 QA 체크리스트 수행:
  - 기능: 요구사항 구현/예외 처리/AC 충족
  - 품질: 네이밍, 기존 패턴 일관성, 불필요 코드
  - 보안: SQL Injection, 민감정보 로그, 입력 검증
  - 데이터: 트랜잭션 경계, N+1, 인덱스
  - 계층: Controller → Service → Repository 준수 (CLAUDE.md 아키텍처 규칙)

## 산출물

`docs/output/{domain}/review-report.md` (도메인 파악 가능한 경우)

## 결과 보고

발견된 이슈를 심각도별(Critical/Major/Minor)로 요약 보고. 리포트 상단에 리뷰 범위 모드와 검토된 파일 수를 명시.

## 사용 예시

```
/review                   # 워킹 트리 변경 리뷰
/review staged            # 스테이징된 변경만 리뷰
/review unpushed          # 푸시 안 한 커밋 전체 리뷰 (PR 직전 추천)
/review unpushed 트랜잭션 경계 꼼꼼히    # 추가 요청 함께 전달
```
