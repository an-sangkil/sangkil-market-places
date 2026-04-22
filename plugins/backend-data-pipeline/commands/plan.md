---
description: 검증된 스펙을 기반으로 기술 개발 플랜 작성 (spec 이후 단계)
---

이미 작성된 스펙/요구사항 문서를 기반으로 **기술 개발 플랜**을 작성한다.

**요청 내용 (feature-name 또는 스펙 경로):**
$ARGUMENTS

## 실행 방식

### 1) 스펙 경로 확인

`$ARGUMENTS`로부터 스펙 경로를 결정한다:

- **feature-name이 주어진 경우**: `doc/output/{feature-name}/requirements.md` 사용
- **파일 경로가 주어진 경우**: 해당 경로 사용
- **인자가 없는 경우**:
  - `ls -lt doc/output/*/requirements.md | head -5`로 최근 스펙 후보 제시
  - 사용자에게 어떤 스펙으로 플랜을 작성할지 확인

스펙 파일이 존재하지 않으면:
```
❌ 스펙 문서를 찾을 수 없습니다.
먼저 /spec 명령으로 스펙을 작성하세요.
```
라고 안내하고 중단.

### 2) development-plan 에이전트 호출

Agent tool로 `development-plan` 에이전트를 호출하며 스펙 경로를 전달한다.

## 산출물

`doc/output/{feature-name}/plan.md`

포함 내용:
- 태스크 목록 (의존성 순서)
- 파일 변경 계획 (신규/수정)
- 데이터 모델 변경 (DDL)
- 검증 계획
- 리스크 및 완료 기준

## 결과 보고

플랜 요약을 사용자에게 보고한 뒤 다음 단계 안내:

```
✅ 개발 플랜 작성 완료
📄 산출물: doc/output/{feature-name}/plan.md

➡️ 다음 단계:
   /build {feature-name}  → TDD로 구현
   /pipeline              → 구현 + QA + 배포까지 자동
```

## 언제 쓰나

- `/spec`으로 요구사항을 확정한 뒤 플랜만 별도로 작성
- 기존 스펙 문서로 플랜만 다시 작성하고 싶을 때
- 구현 전에 플랜을 리뷰받고 싶을 때
