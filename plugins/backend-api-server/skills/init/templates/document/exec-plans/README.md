# document/exec-plans/

기능별 요구사항과 실행계획을 관리하는 디렉토리.

## 파일 구조

```
exec-plans/
├── {domain}/
│   ├── requirements.md      ← /spec 커맨드로 생성
│   ├── plan.md              ← /plan 커맨드로 생성
│   ├── test-report.md       ← /test 커맨드로 생성
│   ├── review-report.md     ← /review 커맨드로 생성
│   └── qa-report.md         ← /qa 커맨드로 생성
```

## 작성 규칙

- 도메인(기능) 단위로 하위 폴더를 생성한다.
- 요구사항 문서에는 `공통 아키텍처: document/architecture/backend-application-architecture.md` 참조를 포함한다.
- 공통 아키텍처 내용을 중복 서술하지 않는다.
