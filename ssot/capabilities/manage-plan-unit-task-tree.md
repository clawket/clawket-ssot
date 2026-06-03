---
id: capability.manage-plan-unit-task-tree
kind: Capability
title: 사이드바 트리에서 플랜·유닛·태스크 관리
definition: 웹 대시보드 사이드바의 계층 트리에서 플랜·유닛·태스크를 추가하고, 제목·상태를 더블클릭 인라인으로 고치며, 플랜 승인과 태스크 일괄 변경(상태·사이클·유닛 이동)을 한 자리에서 수행하는 기능.
purpose: 사람 운영자가 화면을 옮겨다니지 않고 작업 트리 전체를 한눈에 보며 빠르게 손볼 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/PlanTree.tsx
  - web/src/components/CreatePlanModal.tsx
  - web/src/components/CreateUnitModal.tsx
  - web/src/components/CreateTaskModal.tsx
  - web/src/hooks/useInlineEdit.ts
relatesTo:
  - to: concept.plan
    type: mutates
    note: 트리 상단 노드로 표시하고 draft 플랜은 Approve 버튼으로 active 전환
  - to: concept.unit
    type: mutates
    note: 플랜 하위 그룹으로 표시하고 태스크 진척도를 묶어 보여줌
  - to: concept.task
    type: mutates
    note: 최하위 노드로 표시·생성·인라인 편집·일괄 변경의 주 대상
  - to: concept.cycle
    type: reads
    note: 편집 모드 일괄 변경에서 태스크를 사이클에 배정/해제
impacts:
  - concept.plan
  - concept.unit
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

사람 운영자는 사이드바의 플랜 트리에서 작업 계층 전체를 펼쳐 보며 직접 손볼 수 있다. 새 플랜·유닛·태스크를 그 자리에서 만들고, 태스크의 제목이나 상태를 별도 화면으로 들어가지 않고 더블클릭으로 바로 고친다. draft 상태의 플랜은 트리에서 바로 승인해 active 로 올릴 수 있고, 여러 태스크를 한 번에 골라 상태·사이클·소속 유닛을 한꺼번에 바꿀 수 있다. 목적은 작업 트리를 보는 화면과 고치는 화면을 분리하지 않아, 운영자가 맥락을 잃지 않고 빠르게 정리하도록 돕는 것이다.

## 행위

- 트리에서 플랜/유닛/태스크 노드를 펼치거나 접고 선택한다. active 플랜은 처음 로드 시 자동으로 펼쳐진다.
- "New Plan" / 플랜 옆 "+" / 유닛 옆 "+" 로 각각 플랜·유닛·태스크 생성 모달을 연다.
- 태스크 제목을 더블클릭하면 인라인 입력으로 바뀌고, Enter 로 저장, Escape 로 취소한다. 빈 값이거나 기존과 같으면 저장하지 않는다.
- 태스크 상태 아이콘을 더블클릭하면 상태 선택 드롭다운으로 바뀌어 todo/in_progress/blocked/done/cancelled 로 바꾼다.
- draft 플랜의 "Approve" 버튼으로 플랜을 승인한다. 프로젝트에 이미 active 플랜이 있으면 데몬이 거부하고, 화면은 그 사유를 토스트로 알린다.
- "Edit" 모드에 들어가면 태스크마다 체크박스가 생긴다. 여러 태스크를 고른 뒤 하단 액션 바에서 상태 변경, 사이클 배정/해제, 다른 유닛으로 이동을 일괄 적용한다. 전체 선택/해제도 제공한다.
- 각 유닛 옆에는 종료된 태스크 비율을 막대와 "완료/전체" 숫자로 보여준다.

## 시스템 흐름

트리는 활성 프로젝트의 플랜 목록을 읽고, 각 플랜의 유닛과 그 태스크를 함께 불러와 계층으로 구성한다. 인라인 편집·상태 변경·일괄 변경은 모두 태스크 갱신 요청으로 데몬에 전달되며, 성공하면 트리를 다시 불러와 최신 상태를 반영한다. 플랜 승인은 별도의 승인 요청으로 처리되고, "한 프로젝트에 active 플랜은 하나" 같은 규칙 위반은 데몬이 막아 화면에 알림으로 드러난다. 다른 화면에서 일어난 태스크 변경은 실시간 이벤트 패치로 트리에 즉시 반영되어, 전체를 다시 읽지 않고도 해당 노드만 갱신된다.

## 어디에 구현되어 있나

핵심은 사이드바 트리 컴포넌트(`web/src/components/PlanTree.tsx`)이며, 생성은 각 모달(`CreatePlanModal.tsx` / `CreateUnitModal.tsx` / `CreateTaskModal.tsx`)이 담당한다. 더블클릭 인라인 편집의 상태 관리는 전용 훅(`web/src/hooks/useInlineEdit.ts`)이 맡는다. 모든 변경은 웹 대시보드가 데몬 API를 통해 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: 일괄 변경 시 적용되는 라이프사이클 게이트(예: active 사이클 미배정 태스크 시작 제약)가 화면 단에서 어떻게 안내되는지 확인 필요
