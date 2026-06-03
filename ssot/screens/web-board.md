---
id: screen.web-board
kind: Screen
title: Board 화면 (사이클 칸반)
definition: 선택한 사이클의 태스크를 상태별 칸반 컬럼으로 보여주고 드래그앤드롭으로 태스크 상태를 바꾸는 화면. 사람 운영자가 한 스프린트(사이클)의 작업 흐름을 관리한다.
purpose: 사람 운영자가 한 사이클의 태스크를 상태 컬럼으로 보고 드래그로 상태를 옮기며 사이클 자체를 시작/종료한다
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard]
implementedIn:
  - web/src/components/BoardView.tsx
  - web/src/components/board/
consumesApi:
  - endpoint.daemon-cycles
  - endpoint.daemon-tasks
  - endpoint.daemon-events
relatesTo:
  - { to: capability.web-review, type: realizes, note: 칸반 상태 변경 DnD를 제공하는 핵심 뷰 }
  - { to: concept.cycle, type: reads, note: 사이클을 선택·생성·편집하고 planning↔active↔completed 전환 }
  - { to: concept.task, type: reads, note: 태스크를 상태 컬럼에 배치하고 DnD로 상태 변경 }
  - { to: concept.task-status, type: reads, note: todo/in_progress/done/blocked/cancelled 컬럼 매핑 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Board 화면은 하나의 사이클(스프린트)에 속한 태스크들을 상태별 칸반 보드로 펼쳐, 사람 운영자가 작업 흐름을 직관적으로 관리하도록 한다. 어느 작업이 어느 단계에 있는지 컬럼으로 한눈에 보고, 카드를 끌어다 옮기는 것만으로 상태를 바꾼다. 또한 이 화면에서 사이클을 새로 만들고, 편집하고, 시작·종료하는 라이프사이클 제어도 함께 수행한다.

## UI 요소 / 입력 필드

- **사이클 선택 툴바**: 완료되지 않은 사이클을 우선 노출하고 완료 사이클은 구분선 아래에 나열하는 선택 드롭다운. "+ New Cycle" 버튼으로 새 사이클을 만들고, 선택된 사이클에 대해 편집 버튼·상태 배지·상태 변경 드롭다운(planning/active)을 제공한다.
- **사이클 헤더**: 선택된 사이클의 제목과 목표(goal)를 표시.
- **칸반 컬럼**: 정해진 컬럼 구성(todo·in_progress·done·blocked 등)에 각 상태의 태스크 카드를 쌓아 보여주고, 컬럼별 개수를 함께 표시한다. 빈 컬럼은 "No tasks"로 안내.
- **태스크 카드**: 클릭하면 태스크 상세로 이동하고, 카드 자체의 상태 변경 컨트롤도 제공한다. 카드에는 티어 배지 등이 표시된다.
- **드래그앤드롭**: 카드를 다른 상태 컬럼으로 끌어 놓으면 그 태스크의 상태가 해당 컬럼 상태로 변경된다. 드래그 중에는 반투명 오버레이 카드가 따라온다.
- **보관(Archived) 섹션**: 닫힌 상태의 태스크들을 별도 영역으로 모아 보여준다.

## 표시 데이터 / 호출 API

진입 시 프로젝트의 사이클 목록을 불러와, 활성 사이클이 있으면 그것을, 없으면 미완료 사이클 중 하나를 기본 선택하고 해당 사이클의 태스크를 적재한다. 카드를 다른 컬럼으로 옮기면 그 태스크의 상태를 갱신하고 현재 사이클의 태스크를 다시 불러온다. 사이클 상태 변경은 사이클 갱신 API로 처리한다. 데몬의 사이클·태스크 조회/갱신 API를 사용하며, App 상위에서 전달되는 SSE 태스크 델타 버퍼 계약을 받지만 현재 화면 재렌더는 상위 마운트 키로 구동된다.

상태별 분류는 클라이언트에서 태스크를 컬럼별로 나누어 수행한다.

## 상태 / 엣지케이스

- **로딩 중 / 에러**: 각각 "Loading board..." 와 에러 메시지를 표시.
- **사이클 없음**: "No cycles yet" 안내와 함께 새 사이클 생성 버튼만 노출.
- **빈 컬럼**: 컬럼에 "No tasks" 안내.
- **같은 컬럼으로 드롭**: 상태가 동일하면 아무 동작도 하지 않음.
- 사이클 상태 전환 중에는 중복 요청을 막기 위해 컨트롤을 비활성화한다.
- 완료된 사이클은 선택 드롭다운에서 구분선 아래에 분리 표시되며 편집 버튼은 숨겨진다(완료 사이클 재시작 불가 불변식과 정합).

## 미확정 (OPEN)
- [ ] OPEN: 칸반 컬럼의 정확한 구성과 순서(board/constants)를 자연어로 확정
- [ ] OPEN: SSE 델타가 in-place 패치로 전환될 경우의 동작 변화 추적
