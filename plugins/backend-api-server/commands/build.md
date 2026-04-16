---
description: 구현만 수행 (backend-implementation 에이전트 단독 실행)
---

파이프라인 전체를 돌리지 않고 **구현만** 수행한다.

**요청 내용 (도메인명 또는 플랜 경로 또는 요구사항 텍스트):**
$ARGUMENTS

## 실행 방식

### 1) 입력 해석

`$ARGUMENTS`로부터 구현 단서를 결정한다:

- **도메인명이 주어진 경우**: `document/exec-plans/{domain}/plan.md` 읽어 컨텍스트 확보
- **파일 경로가 주어진 경우**: 해당 경로 사용
- **직접 요구사항인 경우**: 텍스트를 그대로 전달하며 플랜이 없음을 에이전트에 알림

### 2) backend-implementation 에이전트 호출

Agent tool로 다음을 전달:
- 도메인명
- 플랜 경로 (있으면)
- API 계약/엔티티 목록 (플랜에서 추출 가능하면)
- 사용자 요청 텍스트

## 에이전트 동작

- `document/architecture/backend-application-architecture.md` + `CLAUDE.md` + 유사 도메인 기존 코드 참조
- 구현 순서: Entity → Repository → Service → Controller
- `./gradlew :karechat-mydata-api:compileJava` 컴파일 검증
- 기존 테스트가 깨지지 않는지 확인
- 구현 후 `api-spec-writing` 핸드오프 정보 반환

## 완료 기준
- 컴파일 성공
- 기존 테스트 미파손
- Controller → Service → Repository 계층 구조 준수
- 생성/수정 파일 경로 목록 반환

## 결과 보고

```
✅ 구현 완료
📁 생성/수정 파일: {파일 목록}

➡️ 다음 단계:
   /qa              → 테스트/검증
   /notion-upload   → 산출물 공유
```
