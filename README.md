# my-claude-marketplace

Backend Claude Code 플러그인 마켓플레이스

## 플러그인 목록

| 플러그인 | 설명 | 키워드 |
|---------|------|--------|
| **backend-api-server** | Spring Boot 백엔드 API 서버 개발 파이프라인 (spec → plan → build → test → review → qa → notion) | spring-boot, backend, pipeline, tdd, qa |
| **backend-data-pipeline** | Spring Boot 이벤트 드리븐 데이터 파이프라인 개발 라이프사이클 (spec → plan → build⇄qa → deploy → handoff → notion) | spring-boot, data-pipeline, event-driven |
| **git-commit-korean** | Git 변경사항을 파일 단위로 검토하고 한글 커밋 메시지로 안전하게 커밋 | git, commit, korean |
| **github-pr-korean** | 현재 브랜치의 전체 커밋을 분석하여 한글 제목과 본문으로 GitHub PR 생성 | github, pull-request, korean |

## 설치 방법

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add kakaohealthcare/my-claude-marketplace

# 2. 원하는 플러그인 설치
/plugin install backend-api-server@my-claude-marketplace
/plugin install git-commit-korean@my-claude-marketplace
```

## 플러그인 구조

각 플러그인은 다음 구성요소를 포함할 수 있습니다:

- **agents/** — 특정 역할을 수행하는 에이전트 정의
- **commands/** — 슬래시 커맨드 (`/플러그인명:커맨드`)
- **skills/** — 재사용 가능한 스킬 및 참조 파일

상세 사용법은 각 플러그인의 `COMMANDS_README.md`를 참고하세요.

## 플러그인 추가 방법

1. `plugins/` 하위에 플러그인 디렉토리 생성
2. 필요한 agents, commands, skills 배치 (자동 발견)
3. `.claude-plugin/marketplace.json`에 플러그인 항목 추가

## 라이선스

MIT
