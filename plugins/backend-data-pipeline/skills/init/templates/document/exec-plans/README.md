# docs/exec-plans/

단건 실행 기록을 관리하는 디렉토리.

## 파일 구조

```
exec-plans/
├── YYYY-MM-DD-제목.md      ← /build 단독 실행 시 생성
```

## 작성 규칙

- 파이프라인 없이 단독 `/build` 실행 시 실행 계획을 기록한다.
- 설계 문서(requirements, plan)는 `docs/design/{domain}/`에 작성한다.
- 요구사항 문서에는 `공통 아키텍처: docs/architecture/data-pipeline-architecture.md` 참조를 포함한다.
- 공통 아키텍처 내용을 중복 서술하지 않는다.
