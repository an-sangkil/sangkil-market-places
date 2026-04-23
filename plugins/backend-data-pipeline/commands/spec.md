---
description: 요구사항 분석 및 스펙(PRD) 작성만 수행 (플랜 작성 전 단계)
---

사용자의 요구사항을 분석하여 **스펙/PRD 문서**를 작성한다. 개발 플랜은 작성하지 않는다.

**요청 내용:**
$ARGUMENTS

## 실행 방식

Agent tool로 `analyze-requirements` 에이전트를 호출한다.

**전달 항목:**
- 사용자 요구사항 텍스트
- API 명세서 (제공된 경우)
- Figma URL (제공된 경우)
- Notion URL (제공된 경우)

## 추가 정보 요청

요구사항이 모호하거나 부족하면 사용자에게 물어본다:
- Figma 디자인 URL
- 기존 Notion 문서
- API 명세서
- 연관된 기존 기능

## 산출물

`docs/design/{feature-name}/requirements.md`

포함 내용:
- 목표 (Objectives)
- 기능 요구사항 / 비기능 요구사항
- 데이터 모델 / 인터페이스 개요
- 경계 (Boundaries) — 이 작업에서 다루지 않는 것
- 완료 기준 (Acceptance Criteria)

## 결과 보고

스펙 요약을 사용자에게 보고한 뒤 다음 단계 안내:

```
✅ 스펙 작성 완료
📄 산출물: docs/design/{feature-name}/requirements.md

➡️ 다음 단계:
   /plan {feature-name}   → 기술 플랜 작성
   /pipeline              → 구현까지 자동 진행
```

## 언제 쓰나

- 구현 전 **요구사항만 확정**하고 리뷰받고 싶을 때
- 기획/PM과 스펙을 검토한 후 플랜을 시작하고 싶을 때
- 요구사항이 아직 불명확해서 플랜 작성이 시기상조일 때
