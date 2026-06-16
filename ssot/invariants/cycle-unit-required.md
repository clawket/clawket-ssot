---
id: invariant.cycle-unit-required
kind: Invariant
title: 사이클은 유닛 안에서만, 태스크는 사이클·유닛 안에서만 존재한다
definition: "사이클을 만들 때 소속 유닛이 비어 있으면 데몬이 거부하고, 태스크를 만들 때 소속 사이클이나 유닛이 비어 있으면 거부한다. 작업 계층은 항상 유닛 ⊃ 사이클 ⊃ 태스크로 닫혀 있어야 한다(PDD A4: Cycle ⊂ Unit)."
governs:
  - concept.cycle
  - concept.unit
  - concept.task
  - endpoint.daemon-cycles
  - endpoint.daemon-units
decidedBy: []
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
relatesTo:
  - to: concept.cycle
    type: relates-to
    note: 사이클은 반드시 유닛에 소속된다
  - to: concept.task
    type: relates-to
    note: 태스크는 반드시 사이클과 유닛에 소속된다
  - to: invariant.cycle-required-for-tasks
    type: relates-to
    note: 태스크 생성에 사이클을 요구하는 같은 계층 제약의 다른 표현
tags: [status:active, "type:invariant"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 OPEN. -->

## 제약

작업 계층은 위에서 아래로 끊김 없이 닫혀 있어야 한다. 사이클(스프린트)은 어딘가의 유닛(작업 묶음)에 반드시 속해야 하고, 태스크(실제 작업 단위)는 어떤 사이클과 어떤 유닛 안에 반드시 들어가 있어야 한다. 그래서 소속 유닛 없이 사이클을 만들려 하거나, 소속 사이클·유닛 없이 태스크를 만들려 하면 데몬이 생성을 거부한다. 이것은 PDD 방법론의 "사이클은 유닛의 부분집합(Cycle ⊂ Unit)"이라는 A4 원칙을 데이터 층위에서 강제한 것이다.

## 깨지면 무슨 일이 일어나나

소속이 비어 있는 사이클이나 태스크가 만들어지면 "고아" 작업이 생긴다. 어느 묶음·어느 스프린트에 속하는지 알 수 없는 작업은 보드·백로그·타임라인 어디에도 일관되게 배치되지 않고, 진행 집계와 완료 cascade(상위 사이클·유닛·플랜 자동 완료)가 추적할 대상을 잃는다. 계층을 항상 닫힌 상태로 유지해야 "이 작업이 무엇의 일부인가"라는 질문에 언제나 답할 수 있다.

## 코드에서 어떻게 강제되나

데스크탑 데이터 계층(`apps/desktop/src/data/api.ts`)의 생성 페이로드 정의가 이 제약을 그대로 반영한다. 사이클 생성 시 소속 유닛은 필수(PDD A4: Cycle ⊂ Unit)이며 데몬이 비어 있으면 `MISSING_UNIT_ID`로 거부한다고 명시한다. 태스크 생성 시에도 소속 사이클이 없으면 `MISSING_CYCLE_ID`, 소속 유닛이 없으면 `UNIT_REQUIRED`로 데몬이 거부하므로 둘 다 필수 필드로 노출된다. 즉 거부 판정의 최종 권한은 데몬에 있고, 프론트는 그 계약을 타입으로 미러링한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — PDD A4 원칙을 SSOT decision 노드로 정리
