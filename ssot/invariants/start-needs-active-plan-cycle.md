---
id: invariant.start-needs-active-plan-cycle
kind: Invariant
title: 태스크 시작에는 활성 플랜과 활성 사이클이 모두 필요
definition: "태스크를 진행 중 상태로 옮기려면 그 프로젝트에 승인된 활성 플랜이 있고, 태스크가 속한 유닛에 활성 사이클이 배정되어 있어야 한다."
governs:
  - concept.task
  - concept.plan
  - concept.cycle
  - domain.hook-enforcement
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
  - daemon/src/repo/tasks.rs
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

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

태스크 하나를 "지금 작업 중"으로 만들려면 두 가지 전제가 동시에 성립해야 한다. 첫째, 그 프로젝트에 승인되어 활성화된 플랜(로드맵)이 존재해야 한다. 둘째, 그 태스크는 활성 상태의 사이클(타임박스 반복)에 배정되어 있어야 한다.

즉 "Task ⊂ Cycle ⊂ Unit ⊂ Plan" 이라는 작업 계층이 위에서부터 모두 살아 있을 때에만 맨 아래의 태스크가 굴러갈 수 있다. 승인된 계획도 없고 진행 중인 사이클도 없는 상태에서 태스크만 단독으로 돌아가는 것을 막는다.

## 깨지면 무슨 일이 일어나나

이 제약이 없으면 아직 승인되지 않은(초안 상태) 계획의 일이 먼저 실행되거나, 어느 사이클에도 속하지 않은 채 "떠 있는" 태스크가 코드를 바꿀 수 있다. 그러면 그 작업이 어떤 스프린트의 어떤 목표에 기여하는지 추적 고리가 끊긴다. 계획→사이클→태스크로 이어지는 책임 사슬이 보장되지 않으면, 작업 우선순위와 일정 관리가 의미를 잃는다.

## 코드에서 어떻게 강제되나

이 제약은 두 지점에서 함께 지켜진다. 첫째, Claude Code의 PreToolUse 훅 핸들러는 진행 중 태스크가 있는지 확인한 다음, 그 프로젝트에 활성 플랜이 하나라도 있는지(없으면 거부), 활성 사이클이 하나라도 있는지(없으면 거부)를 차례로 검사한다. 또한 변경 도구의 경우 진행 중 태스크가 실제로 사이클에 배정되어 있는지(배정이 없으면 거부)까지 본다(`clawket/adapters/shared/claude-hooks.cjs`의 PreToolUse 게이트). 활성 플랜 검사는 데몬 응답을 해석하지 못하는 경우에도 안전하게 차단하는 하드 블록으로 동작한다.

둘째, 데몬 측 저장소 로직은 태스크 생성·이동 시 사이클 소속을 보장한다. 태스크에 사이클이 지정되지 않으면 해당 유닛의 활성 사이클을 찾아 자동으로 묶고, 활성 사이클이 여러 개여서 모호하면 명시를 요구한다(`daemon/src/repo/tasks.rs`의 create 경로). 이렇게 훅 게이트가 "통과 여부"를, 데몬이 "상태 저장의 정합성"을 각각 담당해 같은 불변식을 양쪽에서 지킨다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — Task⊂Cycle⊂Unit⊂Plan 계층 강제를 정한 결정 노드 미확인
