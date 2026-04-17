---
description: 코드 QA 체크리스트만 수행 (테스트 실행 스킵, 빠른 리뷰). 기본/스테이징/미푸시 3가지 모드 지원
---

현재 변경사항에 대해 코드 리뷰만 수행한다. 테스트는 실행하지 않는다.

**요청 내용 (선택):**
$ARGUMENTS

## 리뷰 범위 모드

첫 번째 인자로 모드를 지정할 수 있다. 없으면 **기본**.

| 모드 인자 | 리뷰 범위 | 사용하는 git 명령 | 언제 쓰나 |
|---|---|---|---|
| (없음) / `wip` | 워킹 트리 변경 | `git diff HEAD` | 작업 중 잠깐 점검 |
| `staged` | 스테이징된 변경만 | `git diff --staged` | 커밋 직전 점검 |
| `unpushed` | 푸시 안 한 커밋 전체 | `git diff @{u}` 또는 `git diff origin/<branch>...HEAD` | 푸시/PR 직전 점검 |

## 실행 방식

### 1) 모드 판별 및 안내

`$ARGUMENTS`의 첫 단어를 확인:
- `staged` → **staged 모드**
- `unpushed` → **unpushed 모드**
- 그 외 또는 없음 → **wip 모드 (기본)**

모드 판별 후 사용자에게 한 줄로 안내한다:

```
🔍 리뷰 모드: {wip|staged|unpushed}
   - wip:      워킹 트리 변경 (커밋 안 한 것)
   - staged:   스테이징된 변경만 (커밋 직전)
   - unpushed: 푸시 안 한 커밋 전체 (PR 직전)

   다른 모드: /review staged  또는  /review unpushed
```

안내 후 다음 단계를 진행한다.

### 2) 변경사항 파악

모드별로 다음 명령을 실행한다:

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
스테이징된 파일이 없으면 "스테이징된 변경이 없습니다. `git add <file>` 먼저 실행하세요"라고 안내하고 종료.

**unpushed 모드:**
```bash
git fetch origin --quiet
git log @{u}..HEAD --oneline
git diff @{u}...HEAD --name-only
```
업스트림 대비 커밋이 없으면 "푸시 안 한 커밋이 없습니다"라고 안내하고 종료.
업스트림이 없는 경우 `git rev-parse --abbrev-ref HEAD`로 브랜치명을 확인하고 `origin/main...HEAD` 범위로 대체한다.

### 3) test-qa 에이전트 호출

Agent tool로 다음을 전달:
- **mode: `review`**
- **리뷰 범위 모드**: wip / staged / unpushed
- 변경된 파일 목록
- 개발 플랜 경로 (있으면)

## test-qa 에이전트 동작 (mode=review)

- 테스트 실행 **스킵** (빠름)
- 테스트 품질 리뷰 **스킵**
- 코드 QA 체크리스트만 수행 (기능/코드품질/보안/데이터/파싱)
- 산출물: `doc/output/{feature-name}/review-report.md`

## 결과 보고

발견된 이슈를 심각도별로 요약하여 사용자에게 보고한다.
리포트 상단에 리뷰 범위 모드와 검토된 파일 수를 명시한다.

## 사용 예시

```
/review                    # 워킹 트리 변경(커밋 안 한 것) 리뷰
/review wip                # 위와 동일 (명시적)
/review staged             # git add로 스테이징된 변경만 리뷰
/review unpushed           # 푸시 안 한 커밋 전체 리뷰 (PR 직전 최종 점검)

/review unpushed 특히 트랜잭션 경계 꼼꼼히 봐줘    # 추가 요청사항 같이 전달
```
