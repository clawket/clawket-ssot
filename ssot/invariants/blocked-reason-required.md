---
id: invariant.blocked-reason-required
kind: Invariant
title: 태스크 차단(blocked)에는 차단 사유가 필수
definition: 태스크를 차단(blocked) 상태로 전환하려면 비어 있지 않은 차단 사유가 반드시 있어야 하며, 없으면 데몬이 전환을 거부한다.
governs:
  - concept.task
  - domain.work-management
implementedIn:
  - daemon/src/repo/tasks.rs
  - cli/src/main.rs
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 자연어 우선. 식별자는 provenance/근거에만. -->

## 제약

태스크가 외부 의존 등으로 진행 불가가 되어 차단 상태로 들어갈 때는, "무엇 때문에 막혔는가"를 설명하는 차단 사유가 반드시 함께 있어야 한다. 공백뿐인 값은 사유 없음으로 취급한다. 이 검사는 "차단으로 들어가는" 전환에만 적용된다 — 이미 차단된 태스크를 다시 차단으로 두는 반복 요청은 새 사유를 요구하지 않는다(멱등). 차단으로 진입하는데 사유가 비어 있으면 전환이 거부된다.

## 깨지면 무슨 일이 일어나나

사유 없이 태스크가 차단될 수 있다면, 막힌 작업의 원인이 기록되지 않아 무엇을 풀어야 진행되는지 추적할 수 없다. 차단이 감사 추적에 남지 않은 채 쌓이면, 사람이나 다른 에이전트가 블로커를 해소하거나 우선순위를 판단할 근거를 잃고, 작업 흐름이 조용히 정체된다.

## 코드에서 어떻게 강제되나

데몬의 태스크 상태 전환 로직에서 차단으로 진입할 때 사유의 존재를 검사해, 비어 있으면 거부한다(`daemon/src/repo/tasks.rs`, BLOCKED_REASON_REQUIRED — v3에서 soft-warn이 아니라 하드 400으로 강화됨). 이미 차단된 태스크의 멱등 재차단은 허용하고, 차단으로 새로 들어갈 때만 실패시킨다. CLI 태스크 명령(`cli/src/main.rs`)도 차단 전환 시 사유를 받도록 안내한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 차단 사유 강제를 하드 400으로 올린 결정(FIX-DAEMON-r2-task-state)을 decision 노드로 연결.
