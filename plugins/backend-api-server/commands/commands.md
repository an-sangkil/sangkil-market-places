---
description: 사용 가능한 모든 커맨드 목록과 사용법 보기
---

`.claude/COMMANDS_README.md` 파일을 읽어서 **사용 가능한 커맨드 목록과 사용법**을 사용자에게 보여준다.

## 실행 방식

1. Read 도구로 `.claude/COMMANDS_README.md` 내용을 읽는다
2. 다음 정보를 사용자에게 정리해서 출력한다:
   - 전체 커맨드 표 (커맨드명 / 설명 / 호출 에이전트)
   - 개발 라이프사이클 순서
   - 자주 쓰는 예시 3~5개
3. 마지막에 "상세 사용법은 `.claude/COMMANDS_README.md`를 참고하세요"라고 안내한다

## 출력 예시 형식

```
📋 사용 가능한 커맨드

| Command | 설명 |
|---|---|
| /spec | 요구사항/스펙 작성 |
| /plan | 실행계획 + TODO |
| /build | 구현 단독 실행 |
| /test | 테스트 실행 + 품질 리뷰 |
| /review | 코드 리뷰만 (3가지 모드) |
| /qa | 전체 검증 |
| /notion-upload | API 명세 Notion 업로드 |
| /commands | 이 목록 보기 |

🔄 개발 흐름:
  /spec → /plan → /build → /review unpushed → /test → /qa → /notion-upload

💡 자주 쓰는 예시:
  /spec 건강검진 위험도 판정 로직
  /build diagnostic
  /review unpushed
  /qa

📖 상세: .claude/COMMANDS_README.md
```
