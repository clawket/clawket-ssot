---
id: capability.board-dnd-status
kind: Capability
title: 칸반 보드 드래그앤드롭 상태 변경
definition: 선택한 사이클의 태스크를 칸반 컬럼(Todo·In Progress·Done·Blocked)으로 드래그앤드롭해 상태를 바꾸고, 사이클을 골라 그 사이클의 작업판을 보는 기능.
purpose: 사람 운영자가 한 스프린트(사이클)의 진행 상황을 시각적으로 보고 태스크 상태를 손쉽게 옮길 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/BoardView.tsx
  - web/src/components/board/DroppableColumn.tsx
  - web/src/components/board/TaskCard.tsx
relatesTo:
  - to: concept.cycle
    type: reads
    note: 보드는 항상 하나의 사이클 단위로 태스크를 모아 보여준다
  - to: concept.task
    type: mutates
    note: 카드를 다른 컬럼으로 옮겨 태스크 상태를 변경한다
  - to: concept.tier
    type: reads
    note: 카드에 태스크의 모델 티어 표식을 함께 보여준다
impacts:
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

사람 운영자는 보드 화면에서 사이클(스프린트) 하나를 골라 그 안의 태스크를 상태별 컬럼으로 나뉜 칸반 보드로 본다. 카드를 끌어다 다른 컬럼에 놓으면 태스크 상태가 그 컬럼의 상태로 바뀐다. 상단 도구막대에서 사이클을 전환하거나 새로 만들고, 사이클 자체의 상태(planning/active/completed)도 바꿀 수 있다. 목적은 진행 중인 스프린트의 흐름을 한 화면에서 직관적으로 파악하고 손으로 옮기듯 상태를 갱신하는 것이다.

## 행위

- 사이클 드롭다운에서 작업판을 볼 사이클을 고른다. 진입 시 active 사이클이 있으면 그것을, 없으면 완료되지 않은 사이클을 우선 선택한다. 완료된 사이클은 구분선 아래에 따로 표시된다.
- 태스크 카드를 컬럼(Todo·In Progress·Done·Blocked) 사이로 드래그해 상태를 바꾼다. 같은 컬럼이면 변경하지 않는다.
- 카드 자체의 상태 변경 컨트롤로도 상태를 바꿀 수 있다.
- "New Cycle" 로 사이클을 새로 만들고, 선택한 사이클의 제목·목표를 편집 모달에서 고친다(완료된 사이클은 편집 버튼이 숨겨진다).
- 사이클 상태 드롭다운으로 planning↔active 를 전환한다.
- 보드 하단의 보관 영역에서 종료된(취소 등) 태스크를 별도로 본다.

## 시스템 흐름

보드는 활성 프로젝트의 사이클 목록을 불러와 기본 사이클을 정하고, 그 사이클에 속한 태스크만 모아 상태별 컬럼에 배치한다. 카드를 다른 컬럼에 놓으면 태스크 상태 갱신 요청이 데몬으로 전달되고, 성공하면 현재 사이클의 태스크를 다시 읽어 보드를 갱신한다. 사이클 상태 변경과 생성·편집도 데몬 호출로 처리되며 결과를 보드 상태에 반영한다. 실시간 이벤트 패치는 보드를 다시 마운트하는 방식으로 반영된다.

## 어디에 구현되어 있나

보드 화면 전체는 `web/src/components/BoardView.tsx` 가 구성하고, 드롭 가능한 컬럼은 `web/src/components/board/DroppableColumn.tsx`, 드래그 가능한 태스크 카드는 `web/src/components/board/TaskCard.tsx` 가 담당한다. 드래그앤드롭은 dnd-kit 기반이며 모든 변경은 웹 대시보드가 데몬 API로 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: 드래그로 done 으로 옮길 때 evidence 필수 규칙이 보드 단에서 어떻게 처리/안내되는지 확인 필요
