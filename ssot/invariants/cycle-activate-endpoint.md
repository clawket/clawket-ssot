---
id: invariant.cycle-activate-endpoint
kind: Invariant
title: 사이클 활성화는 전용 명령으로만 — PATCH로 active 전환 금지
definition: "planning 상태의 사이클을 일반 수정(PATCH status=active)으로 활성화하려 하면 데몬이 거부한다. 반드시 전용 활성화 명령(POST /cycles/:id/activate)을 거쳐야 시작 시각이 서버에 기록되며, 한 유닛 안에서 동시에 active일 수 있는 사이클은 하나뿐이다."
governs:
  - concept.cycle
  - endpoint.daemon-cycles
  - endpoint.cycles-activate
  - capability.manage-cycle
decidedBy: []
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
  - desktop/apps/desktop/src/views/BoardView.tsx
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
relatesTo:
  - to: concept.cycle
    type: relates-to
    note: 사이클의 planning→active 전이 경로를 제한한다
  - to: invariant.one-active-cycle-per-unit
    type: relates-to
    note: "유닛당 active 사이클 1개" 제약과 함께 작동한다
tags: [status:active, "type:invariant"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 OPEN. -->

## 제약

사이클(스프린트성 타임박스)을 "이제 시작" 상태로 바꾸는 일은 아무 수정 경로로나 할 수 없다. 계획 중(planning)인 사이클을 일반 수정 요청으로 곧장 활성(active)으로 바꾸려 하면 데몬이 이를 거부하고, 대신 전용 활성화 명령을 쓰라고 안내한다. 이 전용 경로를 거쳐야만 사이클의 "시작 시각"이 서버에 정확히 남는다.

또한 한 유닛 안에서 동시에 진행 중인 사이클은 하나로 제한된다. 활성화 명령은 이미 활성인 사이클에 다시 걸어도 안전하게 멱등 처리되며, 같은 유닛의 이전 사이클을 자동으로 닫지는 않는다 — 필요하면 호출자가 먼저 그 사이클을 완료시켜야 한다. 마찬가지로 완료(complete)도 전용 명령이 정본 경로이며, 완료된 사이클은 되돌려 다시 활성화할 수 없다(새 사이클을 만들어야 한다).

## 깨지면 무슨 일이 일어나나

만약 일반 수정으로 활성화가 허용된다면, 사이클이 active가 되었는데도 "언제 시작했는지"가 비어 있는 상태가 생긴다. 타임라인·기간 집계·번다운 류 지표가 기준점을 잃어 부정확해지고, 같은 유닛에 두 사이클이 동시에 active가 되어 "지금 진행 중인 스프린트가 무엇인가"가 모호해진다. 시작 시각 기록과 단일 활성 사이클 보장이 활성화의 정본 경로를 강제하는 이유다.

## 코드에서 어떻게 강제되나

데스크탑 데이터 계층(`apps/desktop/src/data/api.ts`)은 사이클 수정 페이로드에서 planning 사이클을 `status: "active"`로 PATCH하는 것을 금지한다고 명시하고, 그 대신 활성화 전용 호출(`activateCycle`, POST `/cycles/:id/activate`)을 쓰도록 규정한다. 데몬은 이 PATCH 시도를 "Use POST /cycles/:id/activate" 안내와 함께 거부하며, 활성화·완료는 각각 시작 시각·종료 시각을 서버에서 기록한다. 보드 화면(`apps/desktop/src/views/BoardView.tsx`)도 사이클 상태 변경을 직접 PATCH가 아니라 `activateCycle`/`completeCycle` 전용 호출로 라우팅한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 이 제약을 도입한 결정 기록
