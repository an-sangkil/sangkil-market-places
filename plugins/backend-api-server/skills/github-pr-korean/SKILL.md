---
name: github-pr-korean
description: GitHub PR을 생성하는 표준 절차를 제공한다. 현재 브랜치의 전체 커밋을 분석하여 한글 제목과 본문을 작성하고 gh CLI로 PR을 생성한다. "PR 올려줘", "풀리퀘 만들어줘", "PR 생성", "리뷰 요청" 등 PR 생성 의도가 보이면 이 스킬을 사용한다.
allowed-tools: Bash, Read, Grep, Glob
---

# GitHub PR Korean

## 목적
현재 브랜치의 변경사항을 분석하여 한글 PR을 일관되게 생성한다.
커밋 로그 전체를 반영한 정확한 제목/본문을 작성하고, 누락이나 실수를 방지한다.

## PR 워크플로우
1. 브랜치 상태와 커밋 이력을 확인한다.
2. 미커밋 변경이 있으면 사용자에게 알린다.
3. 전체 커밋의 diff를 분석하여 PR 제목과 본문을 작성한다.
4. 리모트 push 후 `gh pr create`로 PR을 생성한다.
5. PR URL을 보고한다.

## 실행 절차

### 1) 사전 확인
아래 명령을 병렬로 실행한다:
- `git status --short` — 미커밋 변경 확인
- `git branch --show-current` — 현재 브랜치명 확인
- `git log --oneline -20` — 최근 커밋 확인

base 브랜치를 결정한다:
- 사용자가 지정하면 그것을 쓴다.
- 지정하지 않으면 `main`을 기본으로 한다.

### 2) 변경 범위 분석
base 브랜치와의 차이를 확인한다:
```bash
git log --oneline <base>..HEAD
git diff --stat <base>...HEAD
```

- 커밋이 없으면 "base 브랜치와 차이가 없습니다"라고 알리고 중단한다.
- 미커밋 변경이 있으면 "커밋되지 않은 변경이 있습니다. 먼저 커밋하시겠습니까?"라고 물어본다.

### 3) 상세 diff 검토
전체 변경을 파악한다:
```bash
git diff <base>...HEAD
```

모든 커밋의 변경 의도를 종합하여 PR의 핵심 내용을 정리한다.
최신 커밋 하나만이 아니라, base 이후 전체 커밋을 반드시 반영한다.

### 4) 한글 PR 제목/본문 작성

**제목 규칙:**
- 70자 이내
- 핵심 변경을 한 문장으로 표현
- 프로젝트 커밋 컨벤션이 있으면 동일한 유형 접두어 사용 가능 (예: `feat(건강검진): ...`)

**본문 구조:**
```markdown
## 요약
- 핵심 변경사항 1~3줄 불릿

## 변경 내용
- 주요 변경 파일/로직별 설명

## 참고사항
- 리뷰어가 알아야 할 컨텍스트 (선택, 필요할 때만)
```

본문은 왜 변경했는지를 중심으로 작성한다. 코드를 그대로 나열하지 않는다.

### 5) 리모트 push
현재 브랜치가 리모트에 push되어 있는지 확인한다:
```bash
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null
```

- 리모트 트래킹 브랜치가 없거나, 로컬이 리모트보다 앞서 있으면 push한다:
  ```bash
  git push -u origin <branch>
  ```
- push 전 사용자에게 확인을 받는다.

### 6) PR 생성
HEREDOC으로 본문을 전달한다:
```bash
gh pr create --base <base> --title "<제목>" --body "$(cat <<'EOF'
<본문>
EOF
)"
```

PR 생성 후 URL을 보고한다.

### 7) 추가 옵션 처리
사용자가 요청한 경우에만:
- `--reviewer <user>`: 리뷰어 지정
- `--assignee <user>`: 담당자 지정
- `--label <label>`: 라벨 추가
- `--draft`: 드래프트 PR

## 안전 규칙
- 사용자 확인 없이 `git push`하지 않는다.
- `--force` push는 절대 하지 않는다.
- base 브랜치를 임의로 변경하지 않는다.
- PR이 이미 존재하면 알리고 중단한다 (`gh pr list --head <branch>`로 확인).
- 실패 시 원인(인증, 권한, 네트워크)을 명시하고 다음 조치를 제안한다.
