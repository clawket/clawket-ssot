---
id: endpoint.events
kind: Endpoint
title: 데몬 GET /events·/events/replay — 실시간 변경 스트림(SSE)
definition: 엔티티 변경을 SSE로 실시간 푸시하고, 과거 변경 이력을 유한한 SSE 묶음으로 재생해 주는 이벤트 스트림 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/events.rs]
relatesTo:
  - { to: concept.task, type: reads, note: "태스크 등 엔티티 변경을 이벤트로 전파" }
  - { to: concept.cycle, type: reads, note: "사이클 변경도 같은 스트림으로 전파" }
  - { to: concept.run, type: reads, note: "실행(run) 변경도 같은 스트림으로 전파" }
  - { to: concept.audit-log, type: reads, note: "/events/replay는 감사 로그를 과거 이벤트로 재생" }
governedBy: []
impacts: []
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

엔티티(태스크·사이클·실행 등)에 변경이 생길 때마다 그 사실을 끊지 않고 실시간으로 흘려보내는 이벤트 스트림이다. 웹 대시보드가 이 스트림을 구독해 별도 새로고침 없이 화면을 최신 상태로 갱신하고, 명령줄 관찰 기능도 같은 스트림을 듣는다. 함께 제공되는 재생(replay) 경로는 "지금부터의 실시간"이 아니라 "이미 지나간 변경 이력"을 한 차례 쏟아 보여 준 뒤 닫힌다.

## 요청 / 응답

실시간 스트림(`/events`):
- 요청 시 어떤 종류의 엔티티만 받을지(예: 태스크·사이클)를 쉼표로 구분해 좁힐 수 있고, 생략하면 모든 종류를 받는다.
- 응답은 연결이 유지되는 동안 변경이 생길 때마다 하나씩 도착하는 이벤트들이다. 각 이벤트에는 추적용 식별자, 이벤트 종류, 그리고 변경 내용을 담은 본문이 실린다. 끊김 방지를 위해 일정 간격으로 "살아 있음" 신호(ping)를 함께 보낸다.
- 클라이언트가 마지막으로 받은 이벤트 식별자를 함께 보내면 그 위치를 인지하지만, 실시간 채널 자체는 오래된 이벤트를 다시 채워 주지는 않는다(과거 복원은 재생 경로의 몫).

과거 재생(`/events/replay`):
- 요청 시 어떤 엔티티 종류·어떤 특정 엔티티로 좁힐지, 몇 건까지 받을지를 줄 수 있다(건수 미지정 시 기본 상한이 적용된다).
- 응답은 과거 변경 기록을 이벤트 형태로 차례차례 보낸 뒤 스트림을 닫는다. 각 항목은 대상 엔티티·작업 종류·바뀐 필드와 이전/이후 값·수행 주체·발생 시점을 담는다.

## 권한 / 제약

- 실시간 스트림은 메모리 기반 브로드캐스트라 연결 이전의 과거 이벤트를 보존하지 않는다 — 끊겼다 다시 붙으면 그 사이 변경은 실시간 채널로 메워지지 않고, 과거 이력 복원은 재생 경로(`/events/replay`)로 처리한다.
- 재생 경로는 무한 스트림이 아니라 과거 기록을 한 묶음으로 보낸 뒤 종료하는 유한 스트림이다.
- 로컬 데몬에 직접 붙는 호출이므로 로컬에서만 접근 가능하다.

## provenance

이벤트 라우터(`daemon/src/routes/events.rs`)가 실시간 스트림 경로와 재생 경로를 제공한다. 실시간 스트림은 데몬 내부 브로드캐스트 채널을 구독해 엔티티 종류 필터를 적용하고, 재생 경로는 로컬 SQLite의 감사 로그(`concept.audit-log`)에서 과거 기록을 읽어 이벤트로 변환한다. 데몬(`component.daemon`)이 이 라우터를 호스팅하고 웹 대시보드(`component.web-dashboard`)가 소비한다. 코드가 사라져도 "실시간 변경 푸시 + 끊김 방지 ping + 별도 과거 재생"이라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 실시간 스트림이 전파하는 엔티티 종류의 전체 목록과 각 이벤트 본문의 표준 형태는 데몬 이벤트 발행 지점들을 함께 정독해 확정할 것. 본 노드는 스트림 엔드포인트의 의미·제약 범위만 다룬다.
