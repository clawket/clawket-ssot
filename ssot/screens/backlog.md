---
id: screen.backlog
kind: Screen
title: Backlog 화면 (작업 큐 / 사이클 배정)
purpose: "사람 운영자가 사이클에 배정되지 않은 작업과 활성·계획 사이클의 작업을 한 화면에서 보고, 드래그앤드롭으로 작업을 사이클에 배정·해제하며 사이클을 시작·종료한다."
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/views/BacklogView.tsx
  - desktop/apps/desktop/src/views/ViewShell.tsx
  - web/src/components/BacklogView.tsx
consumesApi: [endpoint.daemon-events]
relatesTo:
  - { to: concept.backlog, type: reads, note: "사이클 미배정 작업 집합을 'Backlog' 섹션으로 보여준다" }
  - { to: concept.task, type: reads, note: "작업 행을 우선순위·티어·상태와 함께 보여주고 배정/해제한다" }
  - { to: concept.cycle, type: reads, note: "활성·계획 사이클을 섹션으로 묶고 시작/종료한다" }
  - { to: concept.tier, type: reads, note: "작업 행에 티어 칩을 표시하며 선언↔실행 티어 불일치를 화살표로 드러낸다" }
  - { to: concept.ticket-number, type: reads, note: "작업 행에 티켓 번호를 표시한다" }
  - { to: capability.web-review, type: realizes, note: "백로그→사이클 배정 DnD 관리 능력을 구현하는 화면" }
impacts: [concept.task, concept.cycle]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Backlog 화면은 "아직 일정에 들어가지 않은 작업"과 "현재/예정 사이클에 들어간 작업"을 한 큐 형태로 보여주는 곳이다. 운영자는 여기서 백로그 작업을 끌어다 사이클에 넣거나 빼고, 계획(planning) 사이클을 시작하거나 활성(active) 사이클을 종료한다. 작업은 활성 프로젝트 범위로만 좁혀 보여준다(플랜→유닛→사이클 연결로 소속을 판별).

## UI 요소 / 입력 필드

- 사이클 섹션(여러 개): 완료되지 않은(활성·계획) 사이클마다 한 섹션. 섹션 헤더는 접기/펼치기 토글, 사이클 제목, 완료/전체 작업 수, 사이클 상태 배지를 담는다. planning 사이클에는 "Start Cycle" 버튼, active 사이클에는 "End Cycle" 버튼이 붙는다.
- 작업 행: 우선순위 점(critical/high/medium/low 색), 티켓 번호, 제목, 티어 칩, 상태 배지, 담당자를 한 줄에 보여준다. 사이클 안의 작업 행에는 사이클에서 빼는(×) 버튼, 백로그 작업 행에는 사이클을 고르는 "+ Cycle" 버튼이 붙는다.
- Backlog 섹션: 사이클에 배정되지 않은 작업을 같은 카드 스타일로 모아 보여주며, 접기/펼치기와 항목 수를 표시한다.
- 드래그앤드롭: 작업 행을 끌어 사이클 섹션에 놓으면 그 사이클에 배정되고, Backlog 섹션에 놓으면 미배정으로 돌아간다.

## 표시 데이터 / 호출 API

화면은 플랜·유닛·사이클·태스크 목록을 받아, 활성 프로젝트에 속하는 사이클과 작업만 골라낸다. 작업은 cycle_id가 있으면 해당 사이클 섹션으로, 없으면 Backlog 섹션으로 분류한다. 데이터는 데몬의 실시간 이벤트 스트림(SSE)을 구독하는 공통 데이터 계층을 통해 받는다.

배정/해제는 작업의 사이클 소속을 갱신하는 동작을 호출하고(사이클에 넣기 = cycle 지정, 빼기 = 미배정), 사이클 시작/종료는 보드와 같은 "사이클 시작"·"사이클 완료" 동작을 호출한다.

## 상태 / 엣지케이스

- 로딩/에러 상태: 각각 "Loading data…", 에러 메시지를 표시한다.
- 사이클이 비었을 때: 해당 섹션에 "No tasks in this cycle. Drag from backlog below." 안내.
- 백로그가 비었을 때: "All tasks are assigned to cycles." 안내.
- 배정/해제·시작/종료 실패: 콘솔에 오류를 남기고 화면은 유지된다(낙관적 갱신 실패 시 데이터는 다음 SSE로 정합화).
- 우선순위/티어 값이 미상이면 기본 색/표시로 폴백한다.

## 미확정 (OPEN)
- [ ] OPEN: 우선순위 값 집합(critical/high/medium/low)의 출처와 정렬 규칙을 데몬 측 정의와 대조해 확정할 것
