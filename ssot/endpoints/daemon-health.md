---
id: endpoint.daemon-health
kind: Endpoint
title: 데몬 GET /health
definition: 로컬 clawketd 데몬이 살아 있는지 한 번에 확인하는 생존 신호 엔드포인트. 정상이면 200으로 응답한다.
realizedBy:
  - component.daemon
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: component.daemon
    type: backed-by
    note: 이 엔드포인트를 제공하는 로컬 데몬
  - to: component.install-gate
    type: depends-on
    note: 설치 직후 게이트가 이 엔드포인트를 핑해 데몬 생존을 확인한다
governedBy: []
impacts:
  - component.install-gate
consumedBy:
  - component.install-gate
  - endpoint.hook-session-start
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
데몬이 살아 있는지를 묻는 가장 단순한 확인 통로다. 무언가를 바꾸거나 데이터를 다루지 않고, 단지 "지금 응답할 수 있는 상태인가"만 알려준다. 주로 설치·기동 직후 자동 점검 과정에서, 그리고 세션을 새로 시작할 때 데몬이 정상 가동 중인지 확인하는 용도로 호출된다. 사람이 직접 부르기보다 플러그인 내부 절차가 자동으로 핑하는 내부용 신호다.

## 요청 / 응답
보내는 정보는 없다 — 단순 조회 호출이다. 돌려받는 것은 "정상 가동 중"이라는 성공 신호(200)뿐이다. 데몬이 아직 뜨지 않았거나 응답하지 못하면 호출 측은 일정 시간(기본 약 3초) 안에 답이 오지 않은 것으로 보고 "살아 있지 않음"으로 판단한다. 즉 응답 본문의 내용보다 "정해진 시간 안에 정상 응답이 왔는가"가 의미의 전부다.

## 권한 / 제약
데몬은 로컬 루프백(127.0.0.1)에만 바인딩되므로 같은 기기 안에서만 호출할 수 있고 외부에서는 접근 불가다. 호출 측은 데몬이 실제로 쓰는 포트를 캐시 디렉터리의 포트 파일(`clawketd.port`)에서 읽어 사용한다 — 포트 파일이 없거나 비어 있으면 호출 자체가 성립하지 않고 곧바로 "살아 있지 않음"으로 처리된다. 설치 게이트는 이 핑이 성공해야만 설치 완료 마커를 유효한 것으로 인정하고, 응답이 없으면 마커를 무효화해 다음 세션에서 재설치를 시도하게 한다.

## provenance
호출 측 구현 근거는 `clawket/adapters/shared/claude-hooks.cjs`의 `pingDaemonHealth` (포트 파일에서 포트를 읽어 `127.0.0.1`의 `/health`에 GET, 200이면 생존으로 판정). 응답을 제공하는 주체는 로컬 데몬(component.daemon)이다. 설치 마커 검증 흐름(component.install-gate)이 이 핑을 게이트로 사용한다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 측 `/health` 핸들러의 응답 본문 형태(상태 텍스트·버전 포함 여부)는 데몬 라우터 코드를 정독해 확인 필요. 현 근거는 호출 측(훅)이 200 여부만 본다는 사실까지.
