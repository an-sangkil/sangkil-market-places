---
description: 개발 산출물들을 통합한 인수인계 문서(deliverable.md) 작성
---

개발 단계에서 흩어져 만들어진 산출물들(스펙, 플랜, QA 리포트, 배포 체크리스트, 변경 코드)을 **하나의 인수인계 문서**로 통합한다.

**요청 내용 (feature-name 또는 경로):**
$ARGUMENTS

## 실행 방식

### 1) feature-name 확인

`$ARGUMENTS`로부터 feature-name을 결정한다:

- **feature-name이 주어진 경우**: `docs/design/{feature-name}/`, `docs/output/{feature-name}/` 사용
- **인자가 없는 경우**:
  - `ls -lt docs/design/` 로 최근 작업 후보 제시
  - 사용자에게 어떤 feature에 대한 handoff인지 확인

### 2) 가용 산출물 수집

다음 파일들의 존재 여부를 확인한다:
- `docs/design/{feature-name}/requirements.md` (스펙)
- `docs/design/{feature-name}/plan.md` (개발 플랜)
- `docs/output/{feature-name}/qa-report*.md` (QA 결과)
- `docs/output/{feature-name}/deploy-checklist.md` (배포 체크리스트)
- `git diff` 변경 파일 목록

일부 파일이 없어도 진행한다. 없는 섹션은 "해당 단계를 수행하지 않음"으로 표시.

### 3) doc-handoff-writer 에이전트 호출

Agent tool로 `doc-handoff-writer` 에이전트를 호출하며 수집된 경로들을 전달한다.

## 산출물

`docs/output/{feature-name}/deliverable.md`

포함 섹션:
1. 개요
2. 구현 내용 (변경 파일, 주요 구현 사항)
3. API 명세 (해당하는 경우만)
4. 데이터 모델 변경 (해당하는 경우만)
5. QA 결과 요약
6. 배포 가이드

## 결과 보고

문서 작성 완료 후 사용자에게 다음 안내:

```
✅ 인수인계 문서 작성 완료
📄 산출물: docs/output/{feature-name}/deliverable.md

➡️ 다음 단계:
   /notion-upload   → Notion으로 업로드
```

## 언제 쓰나

- `/build` 단독 실행 후 최종 인수인계 문서가 필요할 때
- 각 단계를 수동으로 돌린 뒤 결과를 통합할 때
- 기존 deliverable.md를 최신 산출물 기준으로 재생성할 때

**참고:** `/pipeline` 전체 실행 시에는 6단계에서 자동으로 호출되므로 별도 실행 불필요.
