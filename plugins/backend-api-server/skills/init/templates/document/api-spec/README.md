# docs/api-spec/

API 엔드포인트 명세를 관리하는 디렉토리.

## 파일 구조

```
api-spec/
├── {domain}/
│   ├── {domain}-{endpoint}-{action}.md
│   └── ...
```

## 작성 규칙

- 파일명: `kebab-case`, `도메인-엔드포인트-행위.md` 형식
  - 예: `diagnostic-report-query.md`, `user-agreement-list.md`
- 구현 코드를 근거로 작성한다 — 추측으로 내용을 채우지 않는다.
- `@JsonIgnore` 어노테이션이 붙은 필드는 명세에서 제외한다.
- 기존 명세 파일이 있으면 덮어쓰지 말고 diff를 먼저 확인한다.
