---
description: 요구사항 분석 및 스펙(PRD) 작성만 수행 (플랜 작성 전 단계)
---

사용자의 요구사항을 분석하여 **스펙/PRD 문서**를 작성한다. 실행계획은 작성하지 않는다.

**요청 내용:**
$ARGUMENTS

## 실행 방식

Agent tool로 `requirements-intake-planner` 에이전트를 호출한다.

**전달 항목:**
- 사용자 요구사항 텍스트
- 기획 문서 경로 (제공된 경우)
- 정책/규정 문서 (제공된 경우)
- 이미지(OCR) (제공된 경우)

## 추가 정보 요청

요구사항이 모호하거나 부족하면 사용자에게 물어본다:
- 기획 문서(markdown) 경로
- 정책/규정 텍스트
- 기획서 이미지
- 연관된 기존 기능/도메인

## 산출물

`document/exec-plans/{domain}/requirements.md`

포함 내용:
- 요구사항 구체화 (목표/성공기준/범위/비범위)
- 기능개발 목록 (근거 포함)
- `requirements-todo-planner` handoff YAML
- 열린 질문

## 결과 보고

스펙 요약을 사용자에게 보고한 뒤 다음 단계 안내:

```
✅ 스펙 작성 완료
📄 산출물: document/exec-plans/{domain}/requirements.md

➡️ 다음 단계:
   /plan {domain}    → 실행계획 + TODO 작성
```

## 언제 쓰나

- 구현 전 **요구사항만 확정**하고 리뷰받고 싶을 때
- 기획/PM과 스펙을 검토한 후 플랜을 시작하고 싶을 때
- 요구사항이 아직 불명확해서 플랜 작성이 시기상조일 때
