---
description: API 명세 마크다운 파일을 Notion 데이터베이스에 업로드
---

`document/api-spec/` 하위 마크다운 파일을 Notion API 문서 데이터베이스에 업로드한다.

**요청 내용 (선택):**
$ARGUMENTS

## 실행 방식

### 1) 업로드 대상 확인

- 사용자가 파일 경로 또는 도메인명을 명시했으면 그대로 사용
- 명시 안 했으면 `document/api-spec/` 하위 마크다운 목록을 보여주고 사용자에게 확인
  ```bash
  ls -lt document/api-spec/*/*.md | head -10
  ```

### 2) notion-api-spec-upload 에이전트 호출

Agent tool로 다음을 전달:
- 대상 파일 경로 (또는 도메인명)
- Notion 데이터베이스 ID: `334d87e517f480c1a1edff63bed7f7e4`

## 에이전트 동작

- 파일 최상위 `# 제목`을 페이지 제목으로 사용
- 기존 페이지 중복 확인 (`notion-search`)
- 중복 시 사용자에게 **update vs create** 확인
- 마크다운 → Notion 블록 변환 (heading/paragraph/list/code/table 등)
- 업로드 후 생성/갱신된 페이지 URL 반환

## 안전 규칙

- `document/api-spec/` 외부 파일은 업로드하지 않는다
- 덮어쓰기 전 반드시 사용자 확인을 받는다

## 결과 보고

```
✅ 업로드 완료
- 파일: {파일 경로}
- 페이지 제목: {제목}
- Notion URL: {페이지 URL}
- 작업: 신규 생성 | 업데이트
```
