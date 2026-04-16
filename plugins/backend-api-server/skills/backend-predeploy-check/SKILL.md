---
name: backend-predeploy-check
description: CI/보안/모니터링 중심 배포 전 검증 체크리스트
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash
---

사용자 입력: $ARGUMENTS

# 배포 전 검증 체크

배포 전 위험을 체크리스트 기반으로 점검한다.
점검 결과를 pass/fail로 명확히 제시한다.

## 작업 흐름

1. CI 상태(빌드/테스트/정적분석) 결과를 확인한다.
2. 보안 체크(입력 검증, 인증/인가, 민감정보 노출) 항목을 점검한다.
3. 모니터링/알람/대시보드 준비 상태를 점검한다.
4. 롤백 기준과 배포 후 관찰 포인트를 정리한다.
5. 최종 Go/No-Go 판단 근거를 제시한다.

## 출력 형식

1. `배포 전 검증 체크리스트`
2. `미해결 리스크`
3. `Go/No-Go 판단`
4. `배포 후 관찰 포인트`

`배포 전 검증 체크리스트` 표:

| id | area | check_item | status | evidence | action |
|---|---|---|---|---|---|

필드 규칙:
- `area`: `ci` | `security` | `monitoring` | `rollback`
- `status`: `PASS` | `FAIL` | `WARN`

## 원칙

- 상태는 모호한 표현 대신 `PASS/FAIL/WARN`으로 명시한다.
- `FAIL` 항목에는 반드시 후속 조치(`action`)를 적는다.
- 배포 허용 여부는 근거와 함께 제시한다.
- 기본 응답 언어는 한국어로 유지한다.
