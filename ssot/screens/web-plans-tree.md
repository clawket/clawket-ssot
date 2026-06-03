---
id: screen.web-plans-tree
kind: Screen
title: 웹 대시보드 플랜 트리(좌측 사이드바)
definition: 웹 대시보드 좌측 사이드바에 항상 떠 있는 플랜→유닛→태스크 계층 트리로, 어느 뷰에 있든 작업 구조를 탐색·편집하고 여러 태스크를 한 번에 처리할 수 있는 영속 작업 패널이다.
purpose: 사람 운영자가 어떤 뷰에서든 플랜·유닛·태스크 구조를 한눈에 보고, 이름·상태를 인라인으로 고치고, 체크박스로 고른 태스크를 일괄 변경하도록 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/PlanTree.tsx
consumesApi:
  - endpoint.daemon-events
relatesTo:
  - to: concept.plan
    type: relates-to
    note: 트리의 최상위 노드가 플랜이며 여기서 승인(approve)할 수 있다
  - to: concept.unit
    type: relates-to
    note: 플랜 아래 유닛이 그룹으로 펼쳐진다
  - to: concept.task
    type: relates-to
    note: 유닛 아래 개별 태스크가 표시·선택·인라인 편집된다
  - to: capability.web-review
    type: realizes
    note: 사이드바 트리 탐색·인라인 편집·일괄 작업이 웹 리뷰 역량의 일부다
impacts:
  - concept.plan
  - concept.unit
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:screen
---

## 화면 목적
사람 운영자가 웹 대시보드의 어느 뷰(Summary·Board·Backlog·Timeline·Wiki)에 있든, 좌측 사이드바에 고정된 이 트리로 작업 구조 전체를 탐색한다. 트리는 플랜을 최상위로, 그 아래 유닛, 다시 그 아래 태스크를 계층으로 펼쳐 보여 준다. 별도 탭 뒤에 숨지 않고 항상 떠 있어, 화면을 바꿔 가며 작업하는 도중에도 현재 어느 플랜·유닛·태스크를 다루고 있는지 잃지 않게 하는 것이 핵심이다. 트리에서 항목을 고르면 본문 영역에 해당 항목의 상세가 연동되어 열린다.

## UI 요소 / 입력 필드
- 플랜·유닛·태스크 노드: 펼치기/접기로 계층을 탐색한다. 플랜 노드에는 상태(draft·active·completed)에 따른 색 강조가 붙고, 태스크 노드에는 상태 아이콘(todo·진행중·완료·차단·취소)과 우선순위 색점이 표시된다.
- 인라인 편집: 항목 이름과 태스크 상태 등을 트리 안에서 바로 고친다(별도 상세 화면으로 이동하지 않아도 됨).
- 플랜 승인 버튼: draft 플랜을 그 자리에서 active로 승인한다.
- 생성 동작: 새 플랜 추가, 특정 플랜 아래 유닛 추가, 특정 유닛 아래 태스크 추가.
- 일괄 선택(체크박스) 모드: 여러 태스크를 체크박스로 골라 전체 선택/해제하고, 고른 태스크에 같은 변경(예: 상태 일괄 변경)을 한 번에 적용한다.

## 표시 데이터 / 호출 API
보여 주는 정보는 프로젝트에 속한 플랜 목록, 각 플랜의 유닛, 각 유닛의 태스크와 그 상태·우선순위, 그리고 유닛별 태스크 개수 요약이다. 펼친 유닛에 대해서만 실제 태스크 목록을 불러와 트리가 비대해지지 않게 한다. 변경 동작은 데몬의 작업 관리 기능을 통해 처리된다 — 플랜 승인, 항목 인라인 수정, 선택한 태스크들의 일괄 갱신 등이다. 다른 곳에서 일어난 태스크 변경은 데몬의 실시간 이벤트 스트림(SSE)으로 전달받아, 트리 전체를 다시 불러오지 않고 해당 태스크만 제자리에서 갱신한다.

## 상태 / 엣지케이스
- 일괄 작업 중에는 갱신 진행 상태를 두어 중복 적용을 막고, 작업이 끝나면 선택을 비운다.
- 실시간 이벤트로 태스크가 삭제되면 트리에서 해당 노드를 제거하고, 갱신·추가되면 제자리 패치로 반영한다.
- 승인·일괄 갱신 등의 작업이 실패하면 오류 토스트로 사용자에게 알린다.
- 완료된 플랜은 흐리게(opacity 낮춤) 표시해 진행 중인 작업과 시각적으로 구분한다.

## 미확정 (OPEN)
- [ ] OPEN: 일괄 작업으로 적용 가능한 변경의 전체 범위(상태 외 우선순위·삭제 등 포함 여부)를 코드에서 확정할 것
- [ ] OPEN: 사이드바 트리가 호출하는 작업 관리 endpoint 의 정확한 매핑(plan/unit/task 각 CLI·HTTP 대응)을 endpoint 노드와 연결할 것
