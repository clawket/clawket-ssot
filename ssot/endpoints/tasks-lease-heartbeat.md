---
id: endpoint.tasks-lease-heartbeat
kind: Endpoint
title: 태스크 lease heartbeat 갱신
definition: "POST /tasks/{id}/lease/heartbeat — 이미 잡고 있는 태스크 점유(lease)의 만료 시각을 연장한다."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/tasks.rs]
relatesTo:
  - { to: concept.task-lock, type: mutates, note: "기존 점유의 만료 시각(TTL)을 갱신한다" }
  - { to: concept.task, type: reads, note: "대상 태스크가 존재하는지 먼저 확인한다" }
governedBy: []
impacts: [concept.task-lock]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
한 세션(보통 헤드리스로 띄운 코딩 에이전트)이 어떤 태스크를 "지금 내가 작업 중"으로 잡아 두는 점유(lease)에는 만료 시각이 붙는다. 작업이 길어져 그 시각이 다가오면, 같은 세션이 이 동작을 호출해 점유 기한을 다시 미래로 밀어 둔다. 이렇게 주기적으로 "아직 살아 있다"는 신호(heartbeat)를 보내야, 점유가 자동으로 풀려 다른 세션에 빼앗기는 일을 막을 수 있다. 반대로 세션이 죽어 신호가 끊기면 점유는 기한 만료로 저절로 풀려, 멈춰버린 에이전트가 태스크를 영원히 붙잡는 사태를 방지한다.

## 요청 / 응답
요청에는 "누가 잡고 있는가"를 가리키는 세션 식별자와, 선택적으로 이번에 연장할 기한 길이를 담는다. 기한을 생략하면 기본값(헤드리스 에이전트 한 번 실행을 덮기에 충분하면서, 죽은 에이전트의 점유가 몇 분 안에 자동으로 풀릴 만큼 짧은 값)이 적용되고, 너무 긴 값은 상한선으로 잘려 한 세션이 태스크를 무한정 붙잡지 못하게 한다. 세션 식별자는 비어 있으면 거부된다.

성공하면 갱신된 점유 정보(어느 태스크를 어느 세션이 언제부터 잡았고, 언제 만료되며, 마지막 신호 시각은 언제인지)를 돌려준다. 실패하는 경우는 두 갈래다. 대상 태스크가 없으면 "찾을 수 없음"으로 응답하고, 점유가 이미 다른 세션에게 넘어갔거나 신호가 도착하기 전에 만료돼 버린 경우에는 "충돌"로 응답해 갱신을 거절한다.

## 권한 / 제약
점유를 실제로 쥐고 있는 바로 그 세션만 기한을 연장할 수 있다. 신호 도착 시점에 점유가 이미 풀렸거나 다른 세션이 차지했다면 갱신은 거절된다 — 점유의 주인이 한 번에 하나뿐이라는 규칙을 흐트러뜨리지 않기 위해서다. 연장 기한에는 양수여야 한다는 제약과 상한선이 적용된다.

## provenance
구현은 데몬의 태스크 라우터(daemon/src/routes/tasks.rs)에 있으며, 점유 획득/해제와 한 묶음으로 정의된다. 점유 상태 자체는 task-lock 개념이 보관한다.

## 미확정 (OPEN)
- [ ] OPEN: 기본 기한·상한선의 구체 수치와 세션 식별자 발급 주체(어느 컴포넌트가 세션 ID를 만들고 신호 주기를 정하는지)는 운영 정책으로 확정 필요.
