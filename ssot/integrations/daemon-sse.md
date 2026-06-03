---
id: integration.daemon-sse
kind: Integration
title: 웹 대시보드 ↔ 데몬 실시간 이벤트(SSE) 연동
purpose: CLI·다른 탭·에이전트가 만든 변경을 폴링 없이 즉시 화면에 반영해, 여러 주체가 같은 상태를 동시에 보고 일하도록 대시보드를 실시간으로 살아 있게 한다.
definition: 데몬이 내보내는 서버 전송 이벤트(SSE) 스트림을 브라우저의 EventSource 로 직접 구독하는 연동. 태스크·유닛·플랜·사이클·지식·코멘트·실행의 생성/수정/삭제 이벤트와 30초 주기 keepalive ping 을 받아, 태스크는 자리에서 갱신(patch)하고 그 외는 구조 새로고침으로 반영하며, 연결 끊김 시 마지막 이벤트 id 로 재연결을 시도한다.
integratesWith:
  - component.web-dashboard
  - component.daemon
implementedIn:
  - web/src/App.tsx
  - web/src/hooks/useRunEvents.ts
  - web/vite.config.ts
impacts:
  - concept.run
  - capability.web-review
relatesTo:
  - to: integration.daemon-http
    type: relates-to
    note: HTTP 가 데이터를 읽고 쓰면, SSE 가 그 변경을 다른 화면들에 즉시 알린다.
  - to: endpoint.daemon-events
    type: backed-by
    note: 구독 대상이 되는 데몬의 이벤트 스트림 엔드포인트.
governedBy:
  - invariant.sse-not-proxied-dev
  - invariant.sse-wire-contract
  - invariant.sse-payload-id-only
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드 옮겨적기 금지(식별자는 provenance/근거에만). -->

## 무엇과 연동하나

웹 대시보드는 데몬이 흘려보내는 **실시간 이벤트 스트림(SSE)** 을 브라우저가 가진 표준 구독 기능으로 직접 듣는다. 목적은 폴링(주기적 재조회) 없이도 화면을 살아 있게 만드는 것이다. CLI 명령, 다른 브라우저 탭, 혹은 작업 중인 에이전트가 무언가를 바꾸면 그 변경이 곧바로 이 스트림으로 흘러나와 현재 보고 있는 화면에 반영된다 — 여러 주체가 같은 상태를 동시에 보며 일할 수 있게 하는 핵심 통로다.

스트림으로 들어오는 신호는 두 종류다.

- **변경 이벤트**: 태스크·유닛·플랜·사이클·지식·코멘트·실행 각각의 생성/수정/삭제. 태스크 변경은 해당 항목만 자리에서 갱신(patch)해 비용을 아끼고, 그 외 엔티티 변경은 트리 구조를 다시 그리는 구조 새로고침으로 처리한다.
- **keepalive ping**: 아무 변경이 없어도 약 30초마다 도착하는 신호. 데이터는 없지만 연결이 살아 있음을 증명해, 화면이 괜히 "지연됨" 상태로 깜박이지 않게 한다.

연결이 끊기면 브라우저가 알아서 재연결하며, 대시보드는 마지막으로 받은 이벤트 id 를 기억해(브라우저 표준 헤더와 로컬 저장 양쪽으로) 재연결 시 그 지점부터 이어받으려 한다.

## 구현 위치 (provenance)

- `web/src/App.tsx` — 메인 SSE 구독부. 데몬이 내보내는 엔티티:변경 조합마다 리스너를 걸어 한 곳의 라우팅 함수로 모으고, 받은 페이로드의 엔티티 종류에 따라 태스크는 자리 갱신, 나머지는 구조 새로고침으로 분기한다. ping 이벤트로 연결 생존을 갱신하고, 마지막 이벤트 id 를 보존해 재연결 시 이어받는다. 연결 상태(연결 중/연결됨/재연결/끊김)를 화면 헤더에 노출한다.
- `web/src/hooks/useRunEvents.ts` — 실행(run) 전용의 좁은 구독 훅. 같은 이벤트 스트림에서 실행 생성/수정만 골라내, 특정 태스크의 실행 관련 화면을 폴링 없이 갱신한다. 페이로드에 태스크 식별자가 없는 이벤트는 추측 없이 버린다.
- `web/vite.config.ts` — 개발 모드에서 이벤트 스트림 경로만 프록시 대상에서 일부러 제외한다. 개발 프록시는 스트림 응답을 버퍼링해 EventSource 가 영원히 연결 대기에 머무는 문제를 일으키므로, 이 경로는 데몬 출처로 직접 붙는다.

사용하는 내부 컴포넌트는 웹 대시보드(`component.web-dashboard`)이고 상대편은 데몬(`component.daemon`)이며, 구독 대상은 데몬의 이벤트 엔드포인트(`endpoint.daemon-events`)다.

## 불변식

- 개발 모드에서 이벤트 스트림 경로는 프록시를 거치지 않고 데몬 출처로 직접 연결해야 한다 — `invariant.sse-not-proxied-dev`. (프록시 버퍼링이 스트림을 멈추게 하므로.)
- 데몬이 내보내는 이벤트 이름·페이로드 형태가 약속된 계약을 지켜야 라우팅이 성립한다 — `invariant.sse-wire-contract`. 다만 일부 옛 데몬을 대비해, 페이로드가 엔티티/변경 종류를 명시하지 않으면 이벤트 이름을 콜론으로 갈라 보강한다.
- 실행 전용 구독은 페이로드 안의 식별자만 신뢰하며, 식별자가 없는 이벤트는 추측하지 않고 버린다 — `invariant.sse-payload-id-only`.

## 영향 범위

실시간 갱신은 칸반·보드 등 대시보드의 협업적 검토 흐름(`capability.web-review`) 전반에 닿는다. 특히 실행 기록(`concept.run`)은 별도 전용 구독으로 화면을 살아 있게 유지하므로 영향 추적의 출발점으로 명시한다. 이 스트림 계약이 바뀌면 모든 뷰의 실시간성이 영향을 받는다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬이 마지막 이벤트 id 부터의 실제 리플레이(누락 이벤트 재전송)를 지원하는 시점 확정 필요
- [ ] OPEN: 태스크 외 엔티티에 대해서도 자리 갱신(patch)을 도입할지 확정 필요
