---
id: capability.backlog-assign-cycle
kind: Capability
title: 백로그 태스크 사이클 배정
definition: 사이클에 배정되지 않은 백로그 태스크를 사이클 영역으로 드래그하거나 드롭다운으로 골라 배정하고, 다시 백로그로 끌어 해제하는 기능. 사이클의 시작·종료도 같은 화면에서 처리한다.
purpose: 사람 운영자가 쌓인 백로그를 어떤 스프린트(사이클)에서 다룰지 일정에 편입·해제하며 계획을 짜게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/BacklogView.tsx
relatesTo:
  - to: concept.backlog
    type: mutates
    note: 미배정 태스크 묶음을 보여주고 거기서 사이클로 끌어낸다
  - to: concept.cycle
    type: mutates
    note: 태스크를 사이클에 넣고 빼며, 사이클 시작/종료도 처리한다
  - to: concept.task
    type: mutates
    note: 태스크의 소속 사이클을 변경한다
impacts:
  - concept.task
  - concept.cycle
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

사람 운영자는 백로그 화면에서 완료되지 않은 사이클들과, 어느 사이클에도 속하지 않은 백로그 태스크 묶음을 함께 본다. 백로그의 태스크를 사이클 영역으로 끌어다 놓으면 그 사이클에 편입되고, 다시 백로그로 끌면 해제된다. 드래그 대신 태스크 옆 메뉴에서 사이클을 직접 골라 배정할 수도 있다. 같은 화면에서 사이클을 시작(planning→active)하거나 종료(active→completed)할 수 있다. 목적은 쌓인 작업을 어느 스프린트에서 처리할지 정리하는 일정 계획이다.

## 행위

- 백로그 태스크를 사이클 영역으로 드래그해 배정하거나, 사이클의 태스크를 백로그 영역으로 드래그해 해제한다. 이미 같은 위치면 아무 일도 하지 않는다.
- 백로그 태스크 옆 "+ Cycle" 메뉴에서 사이클을 골라 배정한다.
- 사이클 안의 태스크 옆 "×" 로 해당 사이클에서 해제한다.
- planning 사이클의 "Start Cycle" 로 active 전환, active 사이클의 "End Cycle" 로 completed 전환한다.
- 각 사이클·백로그 섹션을 접고 펼치며, 사이클마다 종료/전체 진척도를 본다.

## 시스템 흐름

화면은 활성 프로젝트의 사이클 목록과 백로그 태스크를 한꺼번에 읽고, 완료되지 않은 각 사이클의 태스크도 함께 불러온다. 드래그·드롭다운으로 배정/해제하면 해당 태스크의 소속 사이클을 바꾸는 갱신 요청이 데몬으로 가고, 성공하면 전체를 다시 읽어 사이클·백로그 양쪽을 갱신한다. 사이클 시작·종료는 사이클 상태 갱신 요청으로 처리된다. "한 유닛에 active 사이클은 하나" 등의 규칙은 데몬이 강제하며, 화면은 그 결과에 따라 갱신된다.

## 어디에 구현되어 있나

백로그 화면 전체는 `web/src/components/BacklogView.tsx` 한 컴포넌트가 구성하며, 드래그앤드롭은 dnd-kit 기반이다. 배정·해제·사이클 상태 변경은 모두 웹 대시보드가 데몬 API로 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: 사이클 종료(End Cycle) 시 미완료 태스크의 처리(백로그 복귀 여부 등)가 데몬 규칙으로 어떻게 정해지는지 확인 필요
