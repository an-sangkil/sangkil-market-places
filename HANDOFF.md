# 작업 핸드오프 — backend-api-server 플러그인

작성일: 2026-04-16
원본 프로젝트: `/Users/yohan.an/git/mydata/github/mydata-api-server`

## 현재 상태

### 완료된 작업

1. **마켓플레이스 구조 생성** — 공식 스펙 준수
   - `.claude-plugin/marketplace.json` (owner, metadata, source 필드 적용)
   - `plugins/backend-api-server/` 하위에 플러그인 배치
   - `plugin.json` 공식 스펙 준수 (author 객체, 컴포넌트 배열 제거 → 자동 발견)

2. **에이전트 복사 완료** (8개, `plugins/backend-api-server/agents/`)
   - orchestrator, requirements-intake-planner, requirements-todo-planner
   - backend-implementation, test-qa, api-spec-writing
   - backend-docs-sync, notion-api-spec-upload

3. **커맨드 복사 완료** (9개, `plugins/backend-api-server/commands/`)
   - spec, plan, build, test, review, qa, notion-upload, init, commands
   - COMMANDS_README.md 포함

4. **스킬 복사 완료** (3개, `plugins/backend-api-server/skills/`)
   - github-pr-korean, backend-requirements-planning, backend-predeploy-check

5. **템플릿 생성 완료** (`plugins/backend-api-server/templates/document/`)
   - architecture/backend-application-architecture.md — **범용화 적용됨** (모듈명 제거, 네임스페이스 방식)
   - exec-plans/README.md, api-spec/README.md, product-specs/README.md, README.md

6. **범용화 적용 완료**
   - 아키텍처 문서에서 `mydata-model`, `karechat-mydata-api` 등 모듈명 제거
   - 네임스페이스 방식으로 변경 + 멀티모듈 매핑 안내 추가
   - 예시 `diagnostic_report` → `user_account`
   - 에이전트/커맨드의 `document/planning` → `document/exec-plans` 일괄 치환

### 남은 작업 (진행 중단된 지점)

**8번: templates/ → skills/init/ 이동** — 계획 승인 대기 중

변경 계획:
1. `templates/` → `skills/init/templates/` 이동 (공식 스킬 참조 파일 위치)
2. `skills/init/SKILL.md` 신규 생성 (init 스킬 본문)
3. `commands/init.md` 수정 (직접 로직 → init 스킬 호출로 변경)

호출 흐름:
```
/backend-api-server:init
  → commands/init.md (진입점)
    → skills/init/SKILL.md (로직)
      → skills/init/templates/document/* (참조 파일)
```

### 추후 고려 사항

- collector용 플러그인 (`plugins/data-collector-pipeline/`) 추가 가능
- `marketplace.json`에 항목 추가하면 됨
- git repo 초기화는 완료 상태 (main 브랜치, 커밋 미생성)

## 참고: 공식 플러그인 스펙 핵심

- marketplace.json: `.claude-plugin/marketplace.json`에 위치, `owner`(필수), `plugins[].source`(필수)
- plugin.json: `.claude-plugin/plugin.json`, `name`(필수), agents/commands/skills는 기본 경로 자동 발견
- 환경변수: `${CLAUDE_PLUGIN_ROOT}` (플러그인 설치 경로), `${CLAUDE_PLUGIN_DATA}` (데이터 경로)
- 설치: `/plugin marketplace add owner/repo` → `/plugin install plugin-name@marketplace-name`
- 네임스페이싱: `/plugin-name:skill-name` 형식
- references/templates는 비공식 — skills/{name}/ 내부에 배치하면 공식 구조

## 참고: 원본 프로젝트 커밋 이력

| 해시 | 내용 |
|------|------|
| `b9209124` | chore: commands 디렉토리 도입 및 test-qa 에이전트 리팩토링 |
| `08737904` | feat(diagnostic): 건강검진 엔티티 PK에 user_id 포함한 복합키로 변경 |
| `2b24bdcc` | chore: github-pr-korean 스킬 추가 |
