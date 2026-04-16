---
description: 프로젝트에 backend-api-server 플러그인 템플릿을 초기화 (document/ 구조 생성)
---

프로젝트에 **document/ 디렉토리 구조와 아키텍처 템플릿**을 생성한다.

**요청 내용 (선택):**
$ARGUMENTS

## 실행 방식

### 1) 현재 상태 확인

```bash
ls document/ 2>/dev/null || echo "document/ 없음"
```

### 2) 복사 대상 결정

플러그인의 `templates/document/` 하위 파일을 프로젝트 루트의 `document/`로 복사한다.

복사 대상:
- `document/README.md`
- `document/architecture/backend-application-architecture.md`
- `document/exec-plans/README.md`
- `document/api-spec/README.md`
- `document/product-specs/README.md`

### 3) 충돌 처리

- **이미 존재하는 파일은 건너뛴다** (덮어쓰기 없음)
- 건너뛴 파일은 목록으로 보고한다
- 사용자가 `--force`를 명시한 경우에만 기존 파일을 덮어쓴다

### 4) 디렉토리 생성

존재하지 않는 디렉토리는 자동 생성:
```bash
mkdir -p document/{architecture,planning,api-spec,product-specs}
```

### 5) 결과 보고

```
✅ 초기화 완료

생성된 파일:
- document/README.md
- document/architecture/backend-application-architecture.md
- document/exec-plans/README.md
- document/api-spec/README.md
- document/product-specs/README.md

건너뛴 파일 (이미 존재):
- (없음)

➡️ 다음 단계:
   document/architecture/backend-application-architecture.md를 프로젝트에 맞게 수정하세요.
   멀티 모듈이면 패키지 컨벤션 섹션에 모듈 매핑을 추가하세요.
```

## 안전 규칙

- 기본 동작은 **기존 파일 보존** — 사용자가 수정한 파일을 덮어쓰지 않는다
- `document/` 외부에는 아무것도 생성하지 않는다
