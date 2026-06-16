---
id: capability.web-review
kind: Capability
title: 웹 대시보드 리뷰·드래그앤드롭 관리
definition: 사람 운영자가 다섯 개의 화면(Summary·Board·Backlog·Timeline·Wiki)으로 에이전트가 만든 작업 상태를 한눈에 검토하고, 칸반 카드의 상태 변경과 백로그 항목의 사이클 배정을 드래그앤드롭으로 직접 조정하는 기능.
purpose: 사람 운영자가 LLM 에이전트와 동일한 작업 상태를 시각적으로 확인하고 일정·진행을 직접 조율할 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
relatesTo:
  - to: concept.task
    type: mutates
    note: 칸반 카드 드래그로 태스크 상태를 바꾼다
  - to: concept.backlog
    type: mutates
    note: 백로그 항목을 사이클로 끌어 일정에 편입한다
  - to: concept.cycle
    type: reads
    note: 사이클 단위로 백로그를 그룹핑해 보여준다
  - to: concept.knowledge
    type: reads
    note: Wiki 화면에서 지식 항목을 탐색·열람한다
impacts:
  - concept.task
  - concept.cycle
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

사람 운영자는 별도 도구로 옮겨가지 않고도 웹 대시보드 한 곳에서 LLM 에이전트가 진행 중인 작업의 전체 상태를 본다. 진행률·활성 에이전트·유닛별 진척이 요약된 개요부터, 칸반 보드에서의 개별 태스크 상태, 아직 일정에 들어가지 않은 백로그, 에이전트별 실행 타임라인, 그리고 지식 위키까지 다섯 갈래로 나뉜 시점을 자유롭게 오간다.

핵심 가치는 두 가지다. 첫째, 에이전트와 사람이 같은 단일 상태를 본다 — 사람이 보는 화면과 에이전트가 명령줄·MCP로 읽고 쓰는 상태가 어긋나지 않는다. 둘째, 검토에 그치지 않고 마우스로 직접 개입할 수 있다 — 카드를 끌어 상태를 바꾸거나 백로그 항목을 사이클에 배정해 일정을 짠다.

## 행위

- 진행 개요 열람: 프로젝트 전체 진행률, 현재 활동 중인 에이전트, 유닛별 진척을 요약 화면에서 본다.
- 칸반 상태 변경: 보드에서 태스크 카드를 다른 상태 열로 끌어 옮긴다(드래그앤드롭). 이는 해당 태스크의 상태를 바꾸는 행위다.
- 백로그 사이클 배정: 아직 사이클이 없는 백로그 항목을 특정 사이클로 끌어 일정에 편입한다.
- 에이전트 타임라인 관찰: 에이전트별 실행 막대(스윔레인)와 활동 스트림 탭으로 동시 작업 현황을 본다.
- 지식 위키 탐색: 파일 트리 형태로 지식 항목을 열람한다(작성·검색은 별도 위키 지식 기능 영역).

## 시스템 흐름

운영자가 웹 대시보드를 열면 화면은 로컬 데몬의 HTTP API로 현재 상태를 읽어 다섯 뷰로 렌더링하고, 데몬이 푸시하는 실시간 이벤트 스트림을 구독해 태스크·사이클·실행 변화가 생기면 화면이 즉시 갱신된다. 운영자가 카드를 끌어 놓으면 그 변경이 데몬으로 전달되어 상태가 반영되고, 같은 이벤트 스트림을 통해 다른 열린 화면과 에이전트 쪽에도 일관되게 전파된다.

## 어디에 구현되어 있나

이 기능은 React 기반 웹 대시보드(다섯 뷰: Summary·Board·Backlog·Timeline·Wiki)로 실현되며, 상태 읽기·변경은 로컬 데몬의 HTTP API와 실시간 이벤트 스트림에 의존한다. 구체적인 뷰 컴포넌트의 레포 내 경로는 아직 확인되지 않았다.

## 미확정 (OPEN)
- [ ] OPEN: 웹 대시보드 뷰 컴포넌트(Board/Backlog 등)의 clawket 레포 기준 실제 상대경로 확인 필요 (web/ 소스 트리)
- [ ] OPEN: 드래그앤드롭 상태 변경이 호출하는 데몬 변경 엔드포인트의 정확한 경로/메서드 확인 필요 (인벤토리에 해당 mutate 엔드포인트 id 부재)
