---
id: invariant.sse-not-proxied-dev
kind: Invariant
title: 실시간 이벤트 스트림은 개발 모드에서 프록시를 거치지 않는다
definition: 개발 모드에서 실시간 이벤트 스트림(SSE)은 Vite 개발 프록시를 거치지 않고 데몬에 직접 연결되어야 하며, 운영 모드에서는 데몬이 웹을 직접 서빙하므로 같은 출처의 상대 경로로 둔다.
confidence: inferred
lastVerified: 2026-06-04
lifecycle: active
owner: TBD
tags: [status:active, "type:invariant"]
governs: [component.web-dashboard, endpoint.daemon-events]
implementedIn:
  - web/vite.config.ts
  - web/src/lib/daemonUrl.ts
decidedBy: []
dependsOn: [endpoint.daemon-events]
relatesTo:
  - to: component.sse-hook
    type: relates-to
    note: 웹의 SSE 구독 훅이 이 규칙에 따라 dev 에서는 데몬 절대 출처로, prod 에서는 상대 경로로 연결한다.
  - to: invariant.sse-wire-contract
    type: relates-to
    note: 같은 실시간 이벤트 채널의 다른 측면(전송 경로 vs 페이로드 형상) 제약.
crossesBoundary: true
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

웹 대시보드가 데몬의 실시간 이벤트 스트림에 연결하는 경로는 실행 모드에 따라 달라야 한다.

- **개발 모드**: 실시간 이벤트 스트림은 Vite 개발 서버의 프록시를 거치지 않고 데몬의 절대 출처(예: 로컬 데몬 주소)로 직접 연결한다.
- **운영 모드**: 데몬이 웹 번들 자체를 서빙하므로 웹과 이벤트 스트림이 같은 출처에 있다. 따라서 절대 출처를 끼우지 않고 같은 출처의 상대 경로로 연결한다.

이 두 모드를 섞으면 안 된다. 특히 운영 모드에서 절대 출처를 끼워 넣는 것은 금지된다 — 그렇게 하면 같은 출처여야 할 연결이 다른 호스트로의 교차 출처 요청이 되어버린다.

## 깨지면 무슨 일이 일어나나

개발 모드에서 이벤트 스트림을 프록시를 거쳐 연결하면, 개발 프록시가 스트림 조각(이벤트 청크)을 모아 두고 상위 연결이 닫힐 때까지 흘려보내지 않아, 브라우저의 이벤트 수신기가 "연결 중" 상태에 영원히 멈춘다. 실시간 업데이트가 전혀 도착하지 않는다.

반대로 운영 모드에서 절대 출처를 끼워 넣으면, 데몬은 한 주소(루프백)를 광고하는데 브라우저 탭은 다른 이름(localhost)에 있을 수 있어, 한쪽에 묶인 세션 쿠키가 다른 쪽으로 전달되지 않는다. 그 결과 교차 호스트 요청이 되어 실시간 스트림이 통째로 깨진다. 어느 쪽이든 "화면이 상태 변화에 실시간으로 반응한다"는 핵심 경험이 사라진다.

## 코드에서 어떻게 강제되나

빌드 설정(`web/vite.config.ts`)은 개발 모드에서만 데몬 절대 출처를 클라이언트에 주입하고, 운영 빌드에서는 그 값을 빈 문자열로 인라인한다. 또한 개발 프록시 설정에서 이벤트 스트림 경로는 의도적으로 프록시 목록에서 제외되어 있다(주석으로 그 이유가 명시됨). 클라이언트 쪽(`web/src/lib/daemonUrl.ts`)은 주입된 출처가 있으면 그 절대 출처를, 없으면(운영) 빈 문자열을 앞에 붙여 경로를 만든다 — 따라서 운영에서는 자연히 상대 경로로 접힌다. 이 규칙은 웹 빌드 설정과 데몬의 출처/CORS 동작이 함께 맞아야 성립하므로 시스템 경계를 가로지른다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 프록시 우회 결정의 근거 노드 확정 필요
