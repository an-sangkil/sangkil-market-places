---
description: 프로젝트에 fullstack-nextjs-fastapi 플러그인이 가정하는 docs/ 구조, CLAUDE.md, .mcp.json을 초기화한다
---

프로젝트에 **docs/ 디렉토리 구조 + CLAUDE.md + .mcp.json**을 생성한다.

**요청 내용 (선택):**
$ARGUMENTS

## 실행 방식

### 1) 현재 상태 확인

```bash
ls docs/ 2>/dev/null || echo "docs/ 없음"
[ -f CLAUDE.md ] && echo "CLAUDE.md 있음" || echo "CLAUDE.md 없음"
[ -f .mcp.json ] && echo ".mcp.json 있음" || echo ".mcp.json 없음"
```

### 2) 디렉토리 생성

```bash
mkdir -p docs/{design,exec-plans,output,architecture,product-specs}
```

### 3) 템플릿 복사

이 스킬의 `templates/` 하위 파일을 사용자 프로젝트 루트에 복사한다.

복사 대상:
- `CLAUDE.md` — 프로젝트 컨텍스트 템플릿 (도메인/스택 placeholder 포함)
- `.mcp.json` — MySQL MCP 서버 설정 (환경변수 플레이스홀더)

템플릿 원본 경로: `skills/init/templates/`

### 4) 충돌 처리

- **이미 존재하는 파일은 건너뛴다** (덮어쓰기 없음)
- 건너뛴 파일은 목록으로 보고한다
- 사용자가 `--force`를 명시한 경우에만 기존 파일을 덮어쓴다

### 5) 후속 작업 안내

복사 완료 후 사용자에게 다음을 안내한다:

1. **`CLAUDE.md` 편집** — `<<...>>` 으로 감싼 placeholder를 본인 프로젝트 값으로 치환
   - `<<프로젝트명>>`, `<<도메인 한 줄 설명>>`
   - `<<client-dir>>`, `<<server-dir>>` (Next.js / FastAPI 디렉토리명)
   - 도메인 특화 API/테이블 예시는 본인 프로젝트 것으로 교체
2. **`.mcp.json` 환경변수 설정** — `${MYSQL_HOST}` 등의 환경변수를 셸 또는 `.envrc`에 정의
   - 또는 직접 값으로 치환 (단, **절대 git에 커밋 금지**)
3. **`docs/` 디렉토리 사용 패턴** — `/spec`, `/plan`, `/build`, `/qa` 등의 슬래시 커맨드가 자동으로 산출물을 생성하는 위치
   - `docs/design/{feature}/` — 설계 (입력)
   - `docs/exec-plans/` — 단건 실행 기록
   - `docs/output/{feature}/` — 결과물 (검증, 배포, 인수인계)

### 6) 결과 보고

```
✅ 초기화 완료

생성된 디렉토리:
- docs/design/
- docs/exec-plans/
- docs/output/
- docs/architecture/
- docs/product-specs/

복사된 파일:
- CLAUDE.md
- .mcp.json

다음 단계:
1. CLAUDE.md 의 <<placeholder>> 를 본인 프로젝트 값으로 치환
2. .mcp.json 의 ${ENV_VAR} 를 환경변수로 설정 (또는 .envrc 사용)
3. /spec <요구사항> 으로 첫 파이프라인 시작
```

이미 있는 파일은 건너뛴 목록을 함께 표시한다.
