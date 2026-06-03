---
id: screen.board
kind: Screen
title: Board 화면 (활성 사이클 칸반)
purpose: "사람 운영자가 활성 사이클의 작업을 칸반 보드에서 보고, 드래그앤드롭으로 상태를 옮기며, 사이클을 생성·편집·시작·완료한다."
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/views/BoardView.tsx
  - desktop/apps/desktop/src/views/board/constants.ts
  - desktop/apps/desktop/src/views/board/TaskCard.tsx
  - desktop/apps/desktop/src/views/board/DroppableColumn.tsx
  - desktop/apps/desktop/src/views/board/ArchivedSection.tsx
  - web/src/components/BoardView.tsx
  - web/src/components/board/DroppableColumn.tsx
  - web/src/components/board/ArchivedSection.tsx
consumesApi: [endpoint.daemon-events]
relatesTo:
  - { to: concept.cycle, type: reads, note: "보드는 선택된 사이클 단위로 작업을 묶어 보여주고 사이클 상태를 전환한다" }
  - { to: concept.task, type: reads, note: "컬럼은 태스크 상태(todo/in_progress/blocked/done)이며 DnD로 상태를 바꾼다" }
  - { to: concept.plan, type: reads, note: "active 플랜에 속한 사이클만 보드 스코프에 포함한다" }
  - { to: concept.tier, type: reads, note: "작업 카드에 티어를 표시한다" }
  - { to: concept.ticket-number, type: reads, note: "작업 카드에 티켓 번호를 표시한다" }
  - { to: capability.web-review, type: realizes, note: "칸반 DnD 상태 변경 능력을 구현하는 화면" }
  - { to: capability.manage-task-lifecycle, type: realizes, note: "DnD/인라인 버튼으로 태스크 상태 전환을 수행한다" }
impacts: [concept.task, concept.cycle]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Board 화면은 한 사이클(스프린트)에 속한 작업을 칸반으로 펼쳐, 작업을 컬럼 사이로 끌어 옮기며 상태를 바꾸고 사이클 자체를 관리하는 곳이다. 기본으로 active 플랜에 묶인 사이클 중 active 상태인 사이클을 보여주고, 없으면 가장 최근 사이클을 보여준다. 부제에는 선택된 사이클 제목과 작업 수가 나오며, active 플랜은 있는데 사이클이 없으면 "no active cycle", 플랜도 없으면 "No active plan"으로 안내한다.

## UI 요소 / 입력 필드

- 사이클 스코프 바: active 플랜 안의 사이클을 고르는 드롭다운(완료된 사이클은 구분선 아래에 별도 묶음으로 표시), "New Cycle" 버튼, 선택 사이클의 "Edit" 버튼, 상태 배지, 상태 전환 드롭다운.
- 사이클 헤더: 선택된 사이클의 제목과 목표(goal)를 보여준다.
- 칸반 4컬럼: todo · in_progress · blocked · done. 각 컬럼은 드롭 대상이고, 작업 카드를 다른 컬럼으로 끌어다 놓으면 그 상태로 바뀐다. 카드 안에도 인라인 상태 전환 버튼이 있다. 카드는 티켓 번호·티어·제목·담당자·우선순위 점을 담는다.
- 아카이브 섹션: 취소(cancelled)된 작업을 칸반 아래에 따로 모아 보여준다.
- 모달: 사이클 생성 모달, 사이클 편집 모달.

## 표시 데이터 / 호출 API

보드는 플랜·유닛·사이클·태스크 목록을 받아, active 플랜에 속한 유닛의 사이클만 스코프로 잡고 선택 사이클에 묶인 작업만 컬럼별로 분류해 보여준다. 데이터는 데몬에서 실시간 이벤트 스트림(SSE)으로 흘러오며, 데스크탑은 직접 API를 호출하지 않고 공통 데이터 계층(DataProvider)을 통해 받아 다른 화면과 동기 상태를 유지한다.

상태 변경 동작은 데몬을 통한다. 작업을 다른 컬럼으로 옮기거나 인라인 버튼을 누르면 태스크 상태가 갱신된다. 사이클 상태 전환은 의미에 따라 경로가 다르다 — planning 사이클을 active로 만들 때는 단순 상태 패치가 아니라 별도 "사이클 시작" 동작을 호출해 시작 시각이 서버에서 기록되게 하고, 완료는 "사이클 완료" 동작을 호출한다.

## 상태 / 엣지케이스

- 로딩/에러 상태: 각각 "Loading data…", 에러 메시지를 표시한다.
- 사이클 없음: active 플랜에 사이클이 하나도 없으면 "No cycles yet. Create one to start a sprint." 안내와 함께(생성 가능하면) "New Cycle" 버튼을 보여준다. 플랜도 없으면 "No active plan."
- 컬럼 비었을 때: "No tasks"를 표시한다.
- 완료된 사이클: 상태 전환 드롭다운이 비활성화되고, 편집 버튼이 숨겨진다(완료 사이클은 재시작 불가).
- 상태 변경 실패: 보드 상단에 액션 에러 배너를 띄운다.
- 사이클 생성 가능 조건: 활성 프로젝트와 대상 유닛이 모두 있어야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 컬럼 정의(constants.ts)의 색/순서와 web 보드의 정확한 차이를 자연어로 대조할 것
