---
id: endpoint.events-sse
kind: Endpoint
title: 실시간 이벤트 스트림(SSE) 구독
definition: "웹 대시보드가 데몬의 이벤트 스트림에 붙어 엔티티 변경·실행(run)·keepalive(ping)를 실시간으로 받아 폴링 없이 화면을 갱신하고, 끊겼다 재연결될 때 마지막 수신 지점부터 누락분을 재생(replay)받는 구독 엔드포인트."
realizedBy:
  - component.daemon
implementedIn:
  - web/src/App.tsx
  - web/src/hooks/useRunEvents.ts
  - web/vite.config.ts
relatesTo:
  - to: endpoint.daemon-events
    type: reads
    note: 웹은 데몬이 제공하는 /events SSE 스트림을 클라이언트로서 구독한다
  - to: concept.run
    type: reads
    note: run:created/run:updated 이벤트로 실행 상태 변화를 받는다
  - to: concept.task
    type: reads
    note: task/cycle/unit/plan/knowledge/comment 변경 이벤트로 보드·타임라인을 갱신한다
consumedBy:
  - component.web-dashboard
governedBy: []
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
웹 대시보드가 서버를 반복해서 묻지(polling) 않고도, 데이터가 바뀌는 즉시 화면을 새로 그리기 위해 데몬의 단방향 이벤트 스트림(Server-Sent Events)에 구독하는 통로다. 브라우저가 한 번 연결을 열어 두면 데몬이 변화가 생길 때마다 이름이 붙은 이벤트를 밀어 준다. 다루는 이벤트는 크게 세 부류다: 엔티티 변경(태스크·사이클·유닛·플랜·지식·코멘트의 생성/수정/삭제 등), 실행 기록 변화(run 생성·갱신), 그리고 연결 생존을 알리는 keepalive(ping).

## 요청 / 응답
- **연결(요청)**: 스트림 경로에 붙는다. 페이지를 처음 열 때, 이전에 받아 둔 "마지막 이벤트 식별자"가 있으면 그 값을 함께 실어(`?last_event_id=`) 보낸다. 이렇게 하면 데몬이 그 지점 이후에 발생한 이벤트부터 다시 보내 줘서, 연결이 잠시 끊긴 동안 놓친 변화를 메울 수 있다(replay).
- **수신(응답)**: 이벤트마다 "무엇이(엔티티 종류) 어떻게(생성/수정/삭제 등) 바뀌었는지"와 해당 엔티티 식별자가 담긴다. 웹은 받은 이벤트 종류에 따라 보드·타임라인·실행 패널 등 해당 화면 조각만 갱신한다.
  - 실행 관련 구독(useRunEvents)은 특정 태스크로 좁힐 수 있어, 그 태스크에 속한 실행 이벤트만 골라 받는다. 태스크 식별자가 없는 이벤트는 추측하지 않고 그냥 버린다.
  - ping 이벤트는 데이터가 아니라 "연결이 살아 있다"는 신호다. 약 30초 간격으로 오며, 이걸 받으면 "지연(lagging)" 표시가 함부로 켜지지 않도록 생존 시계를 갱신한다.

## 권한 / 제약
- 로컬 데몬과 같은 출처(origin)에서 동작하는 웹에서만 구독한다 — 데몬이 로컬 전용이라 외부 노출이 없다.
- 개발 환경에서는 주의점이 있다: Vite 개발 프록시가 이벤트 스트림(text/event-stream)을 버퍼링해 실시간성이 깨지므로, `/events`만은 프록시를 거치지 않고 데몬 출처로 직접 붙도록 의도적으로 우회한다(`daemonUrl`). 운영 빌드에서는 웹과 데몬이 같은 출처라 우회 없이 그대로 붙는다.
- 끊김 복구는 두 겹이다: 브라우저의 기본 EventSource가 끊기면 알아서 재연결하면서 마지막 이벤트 식별자를 헤더로 보내고, 페이지를 새로 연 경우에는 저장해 둔 식별자를 쿼리로 실어 보내 누락분을 메운다. 연결 상태는 connecting/connected/reconnecting/disconnected로 사람에게 표시된다.
- 지연 판정 임계값은 keepalive 주기(약 30초)보다 넉넉히 커야 한다 — 그렇지 않으면 데몬이 한가할 때 ping 사이마다 배지가 "지연"으로 깜빡인다.

## provenance
웹의 주(主) 구독은 `web/src/App.tsx`(엔티티 변경 전반 + ping + 마지막 이벤트 식별자 저장/복원)이고, 실행 전용의 좁힌 구독은 `web/src/hooks/useRunEvents.ts`다. 개발 프록시 우회 설정은 `web/vite.config.ts`에 있다. 스트림 자체를 제공하는 서버 측은 데몬의 GET /events(endpoint.daemon-events)이며, 웹은 그 클라이언트다. 데몬이 이벤트 이름·페이로드 형태를 바꾸더라도 "이름 붙은 이벤트로 무엇이 어떻게 바뀌었는지를 밀어 주고, last_event_id로 재생을 지원한다"는 규약만 유지되면 웹을 재구현할 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 events 라우터 정독으로 이벤트 식별자(id) 발급 규칙과 last_event_id 재생의 보존 범위(어디까지 거슬러 재생되는지)를 자연어로 확정할 것.
