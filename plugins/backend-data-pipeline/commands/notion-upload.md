---
description: 결과물 MD 파일을 Notion 페이지로 업로드 (notion-upload 에이전트)
---

파이프라인 산출물을 Notion 페이지로 업로드한다.

**요청 내용 (선택):**
$ARGUMENTS

**실행 방식:**

1. **업로드 대상 확인:**
   - 사용자가 경로를 명시했으면 그대로 사용
   - 명시 안 했으면 `doc/output/` 아래 최근 수정된 `deliverable.md`를 찾아 사용자에게 확인
   - `ls -lt doc/output/*/deliverable.md | head -5`로 후보 제시

2. **Notion 위치 확인:**
   - 사용자에게 업로드할 Notion 위치를 물어본다
   - 페이지 URL 또는 검색 키워드 중 하나 입력받음

3. **notion-upload 에이전트 호출** — Agent tool로 다음을 전달:
   - 결과물 MD 경로
   - Notion 위치 정보 (URL 또는 검색 키워드)

**결과 보고:**
업로드 완료 후 **생성된 Notion 페이지 URL**을 사용자에게 전달한다.
