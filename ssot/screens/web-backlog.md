---
id: screen.web-backlog
kind: Screen
title: Backlog 화면 (사이클 배정)
definition: 사이클별 태스크 묶음과 미배정 백로그를 함께 보여주고, 드래그앤드롭으로 태스크를 사이클에 넣거나 빼서 일정을 편성하는 화면.
purpose: 사람 운영자가 미배정 백로그 태스크를 드래그로 사이클에 배정하거나 사이클에서 빼며 스프린트 범위를 편성한다
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard]
implementedIn:
  - web/src/components/BacklogView.tsx
consumesApi:
  - endpoint.daemon-cycles
  - endpoint.daemon-tasks
relatesTo:
  - { to: capability.web-review, type: realizes, note: 백로그→사이클 배정 DnD를 제공하는 뷰 }
  - { to: concept.backlog, type: reads, note: 사이클 미배정 태스크 집합을 표시 }
  - { to: concept.cycle, type: reads, note: 사이클 섹션과 시작/종료 제어 }
  - { to: concept.task, type: reads, note: 태스크의 cycle_id를 DnD로 변경 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Backlog 화면은 아직 일정에 들어가지 않은 태스크(백로그)와 각 사이클이 가진 태스크를 같은 화면에 세로로 쌓아 보여주어, 사람 운영자가 스프린트 범위를 편성하도록 돕는다. 핵심 동작은 태스크를 백로그에서 사이클로(또는 사이클에서 백로그로) 끌어다 옮겨 사이클 편입/해제를 정하는 것이다. 사이클의 시작·종료도 이 화면에서 함께 제어한다.

## UI 요소 / 입력 필드

- **사이클 섹션들**: 완료되지 않은 각 사이클을 카드 형태 섹션으로 보여주고, 헤더에 제목·완료개수/전체개수·상태 배지를 표시한다. 접고 펼칠 수 있으며, planning 사이클에는 "Start Cycle", active 사이클에는 "End Cycle" 버튼이 붙는다. 섹션 안에는 그 사이클에 배정된 태스크들이 행으로 나열되고, 각 행 끝의 × 버튼으로 사이클에서 즉시 제외할 수 있다.
- **백로그 섹션**: 사이클에 배정되지 않은 태스크들을 같은 카드 스타일로 모아 보여준다. 접기 가능하며 항목 수를 표시한다. 각 백로그 행에는 "+ Cycle" 버튼이 있어 드롭다운으로 대상 사이클을 골라 배정할 수 있다.
- **태스크 행**: 우선순위 점 색상, 티켓 번호, 제목, 티어 배지, 상태 배지, 담당자를 표시하고 클릭하면 상세로 이동한다.
- **드래그앤드롭**: 태스크 행을 사이클 섹션 위로 끌어 놓으면 그 사이클로 배정되고, 백로그 영역으로 끌어 놓으면 배정이 해제된다. 드래그 중에는 요약 오버레이가 따라온다.

## 표시 데이터 / 호출 API

진입 시 프로젝트의 사이클 목록과 백로그(미배정 태스크) 목록을 함께 불러오고, 완료되지 않은 각 사이클의 태스크를 추가로 적재한다. 배정/해제는 해당 태스크의 cycle_id를 갱신하는 방식으로 이뤄지며(사이클로 배정 시 대상 사이클 id, 백로그로 이동 시 null), 변경 후 전체를 다시 적재한다. 사이클 시작/종료는 사이클 상태 갱신으로 처리한다. 데몬의 사이클·태스크·백로그 조회/갱신 API를 사용한다.

## 상태 / 엣지케이스

- **로딩 중 / 에러**: 각각 "Loading..." 와 에러 메시지 표시.
- **빈 사이클**: "No tasks in this cycle. Drag from backlog below." 안내.
- **빈 백로그**: "All tasks are assigned to cycles." 안내.
- **무의미한 드롭**: 이미 백로그에 있는 태스크를 다시 백로그로, 또는 같은 사이클로 드롭하면 아무 변화도 일으키지 않는다.
- 완료된 사이클은 섹션 목록에서 제외되어 배정 대상이 되지 않는다(완료 사이클 재시작 불가와 정합).

## 미확정 (OPEN)
- [ ] OPEN: 우선순위(priority) 값 체계와 정렬 규칙을 자연어로 확정
- [ ] OPEN: 데몬 백로그 조회 API(listBacklog)의 정확한 경로를 endpoint 노드로 매핑
