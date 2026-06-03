---
id: endpoint.board
kind: Endpoint
title: 보드(칸반) 대시보드 페이로드 조회
definition: "GET /dashboard/board — 태스크를 상태별 열(todo·in_progress·blocked·done 등)로 묶은 칸반 보드 화면용 데이터를 데몬에서 가져온다."
realizedBy:
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.task
    type: lists
    note: 태스크를 상태(todo/in_progress/blocked/done/cancelled)별로 묶어 열 단위로 나열한다.
  - to: concept.cycle
    type: reads
    note: 보드는 보통 현재 활성 사이클을 기준 범위로 삼아 그 사이클에 속한 태스크를 보여준다.
governedBy: []
impacts: []
consumedBy:
  - component.web-dashboard
  - endpoint.cli-web-views
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

보드 뷰는 태스크를 진행 상태에 따라 세로 열로 나눠 보여주는 칸반 화면이다. 한 열은 "할 일", 다른 열은 "진행 중", 또 다른 열은 "막힘", "완료" 같은 식으로 상태마다 한 칸을 차지하고, 각 태스크 카드가 자기 상태의 열에 놓인다. 사람 운영자가 무엇이 지금 진행 중이고 무엇이 막혀 있는지를 한눈에 파악하거나, 카드를 다른 열로 끌어다 상태를 바꾸는 작업의 바탕이 되는 화면이다. 이 노드는 그 보드를 그리기 위한 데이터를 데몬에서 받아오는 조회 경로를 가리킨다.

조회 단위는 하나의 프로젝트이며, 프로젝트를 지정하지 않으면 현재 작업 디렉터리에 연결된 프로젝트를 대상으로 한다.

## 요청 / 응답

요청에는 어느 프로젝트의 보드를 볼지(생략 시 현재 디렉터리 프로젝트)를 보낸다.

응답은 상태별로 묶인 태스크 묶음이다. 각 태스크 항목은 화면 카드 한 장에 해당하는 정보 — 제목, 사람이 읽는 티켓 번호, 담당자, 현재 상태 — 를 담는다. 보드 화면은 이 묶음을 받아 상태마다 한 열을 만들고 카드를 배치한다.

## 권한 / 제약

로컬 데몬에 연결 가능한 같은 기기의 사용자·에이전트가 조회할 수 있는 읽기 전용 경로다. 보드 화면에서 카드를 드래그해 상태를 바꾸는 행위는 이 조회 경로가 아니라 별도의 태스크 변경 경로를 통해 일어난다(이 노드는 화면을 채우는 읽기만 담당).

## provenance

CLI 쪽 진입점은 `cli/src/main.rs`의 보드 보기 명령으로, 데몬에 보드 대시보드 데이터를 요청해 출력한다. 데이터를 소유하고 상태별로 태스크를 묶어 응답하는 주체는 데몬(component.daemon)이며, 칸반 렌더링은 웹 대시보드(component.web-dashboard)가 맡는다.

## 미확정 (OPEN)
- [ ] OPEN: CLI가 호출하는 `/dashboard/board` 경로에 대응하는 데몬 라우트가 현재 등록되어 있지 않다(데몬은 `/dashboard` 단일 경로만 가짐). CLI의 보드 단축 명령이 데이터를 직접 받는지 웹 뷰만 여는지, 그리고 보드 페이로드를 실제로 만들어 주는 데몬 핸들러 위치를 owner 확인 필요.
- [ ] OPEN: 보드가 보여주는 태스크의 범위(활성 사이클 한정인지, 프로젝트 전체인지, 백로그 포함 여부)와 상태 열 구성의 정확한 규칙은 코드 정독으로 보강 필요.
