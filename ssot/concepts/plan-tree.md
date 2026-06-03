---
id: concept.plan-tree
kind: Concept
title: PlanTree (계획 트리)
definition: 계획(Plan)→단위(Unit)→작업(Task)의 3단 계층을 사이드바에 펼쳐 보여주는 트리 모델로, 사이클(Cycle)은 단위를 가로지르는 개념이라 트리의 행으로는 나타나지 않는다.
relatesTo:
  - to: concept.plan
    type: reads
    note: 트리의 최상위 행(루트)이 계획이다.
  - to: concept.unit
    type: reads
    note: 계획의 자식 행이 단위다.
  - to: concept.task
    type: reads
    note: 단위의 자식 행이 작업이며 단위의 진행률은 자식 작업으로 계산된다.
  - to: concept.cycle
    type: relates-to
    note: 사이클은 단위를 가로지르는 타임박스라 트리 행에 포함되지 않는다.
governedBy: []
dependsOn:
  - concept.plan
  - concept.unit
  - concept.task
implementedIn:
  - desktop/apps/desktop/src/data/planTree.ts
  - desktop/apps/desktop/src/shell/selection.tsx
  - desktop/packages/ui/src/components/PlanTree/PlanTree.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance에만). 분기/의도 불명은 OPEN. -->

## 정의

PlanTree는 데스크탑 사이드바에 작업 구조를 펼쳐 보여주는 트리 모델이다. 계획이 트리의 루트(최상위 행)가 되고, 그 아래 단위가 자식으로, 다시 단위 아래 작업이 손자로 매달린다. 즉 "계획 → 단위 → 작업"의 3단 계층이 한 그루의 나무처럼 표현된다.

트리는 살아 있는 데이터(현재 떠 있는 계획·단위·작업)로부터 만들어지며 정렬 규칙이 정해져 있다 — 계획은 만들어진 순서(오래된 것 먼저), 단위와 작업은 각자의 순서 번호대로 늘어선다. 작업은 자신의 단위 식별자가 일치하는 단위 아래로 묶인다. 각 단위 행에는 진행률(완료된 작업 수 / 전체 작업 수)이 함께 계산되어 표시되는데, 여기서 "완료"로 치는 상태는 done과 cancelled 두 가지다. 계획 행의 진행률은 그 아래 모든 단위의 합으로 집계된다.

**중요한 설계 결정**: 사이클은 이 트리의 행으로 나타나지 않는다. 사이클은 단위에 묶인 타임박스(스프린트)이지만, 트리의 위계상 단위와 작업 사이에 끼어드는 항목이 아니라 단위를 가로질러 작업을 묶는 별개 축이기 때문이다. 그래서 계획 트리는 사이클을 빼고 계획·단위·작업 세 종류의 행만으로 구성된다. 사이클 중심의 작업 뷰는 칸반 보드 화면이 담당한다.

트리의 한 행을 고르면 현재 선택 상태가 갱신되어 상세 드로어가 열리고, 명령 팔레트의 점프 대상과도 연결된다.

## 엔티티 (DB)

PlanTree는 저장되는 엔티티가 아니라 화면 표현용 파생 구조다. 데몬이 내려준 계획·단위·작업 목록을 받아 계층·정렬·진행률을 계산해 트리 노드 숲(forest)으로 조립한다. 각 노드는 종류(계획/단위/작업), 식별자, 표시 레이블, 티켓 번호(있으면), 작업의 경우 상태, 단위·계획의 경우 진행률을 갖는다. 데이터 자체의 단일 진실은 데몬의 계획·단위·작업이고, 트리는 그 위의 읽기 전용 뷰다.

## API 표면

PlanTree 전용 엔드포인트는 없다. 트리는 이미 받아 둔 계획·단위·작업 목록을 클라이언트에서 조립해 만든다.

## 불변식

- 트리 행은 계획·단위·작업 세 종류뿐이며, **사이클은 트리 노드로 포함되지 않는다**(단위를 가로지르는 개념이므로).
- 정렬은 데몬이 준 순서 기준을 따른다 — 계획은 생성 순, 단위·작업은 각자의 순서 번호 순.
- 단위·계획의 진행률 집계에서 done과 cancelled를 "닫힌(완료로 간주되는)" 작업으로 함께 센다.

## 구현 위치 (provenance)

- 계획·단위·작업으로부터 트리를 조립하는 로직: `desktop/apps/desktop/src/data/planTree.ts`
- 트리 행 선택 → 선택 상태/상세 드로어 연결: `desktop/apps/desktop/src/shell/selection.tsx`
- 트리 렌더링 컴포넌트(공유 디자인 시스템): `desktop/packages/ui/src/components/PlanTree/PlanTree.tsx`

## 미확정 (OPEN)
- [ ] OPEN: 단위에 속하지 않은(또는 단위가 없는) 작업이 트리에서 어떻게 처리되는지 확인 필요.
