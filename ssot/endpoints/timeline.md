---
id: endpoint.timeline
kind: Endpoint
title: 타임라인 대시보드 페이로드 조회
definition: "GET /dashboard/timeline — 프로젝트의 태스크·사이클 활동을 시간순으로 늘어놓은 타임라인 화면용 데이터를 데몬에서 가져온다."
realizedBy:
  - component.daemon
implementedIn:
  - cli/src/main.rs
  - daemon/src/routes/timeline.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 태스크의 상태 전이·실행 이력을 시간순으로 읽어 한 줄씩 늘어놓는다.
  - to: concept.cycle
    type: reads
    note: 사이클의 활성화·완료 시점을 같은 타임라인 위에 함께 배치한다.
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

타임라인 뷰는 한 프로젝트 안에서 일어난 작업의 흐름을 시간 축 위에 늘어놓아 보여주는 화면이다. "언제 어떤 태스크가 시작·완료되었고, 어느 사이클이 돌아가고 있었는지"를 한눈에 따라가는 용도다. 이 노드는 그 화면을 그리기 위해 필요한 데이터를 데몬에서 받아오는 조회 경로를 가리킨다. 사람 운영자가 웹 대시보드의 Timeline 탭을 열 때, 그리고 CLI의 타임라인 보기 단축 명령이 화면을 띄울 때 이 데이터가 쓰인다.

조회 단위는 항상 하나의 프로젝트다. 프로젝트를 따로 지정하지 않으면 현재 작업 디렉터리에 연결된 프로젝트를 기본 대상으로 삼는다.

## 요청 / 응답

요청에는 어느 프로젝트의 타임라인을 볼지(생략 시 현재 디렉터리 프로젝트), 얼마나 많은 항목을 가져올지(개수 상한과 건너뛸 개수), 그리고 어떤 종류의 활동만 볼지(예: 태스크만, 사이클만) 거를 조건을 함께 보낼 수 있다.

응답은 시간순으로 정렬된 활동 항목의 목록이다. 각 항목은 "무엇에 대한 사건인지(태스크인지 사이클인지), 어떤 변화였는지, 언제 일어났는지"를 담는다. 화면은 이 목록을 위에서 아래로 또는 좌에서 우로 늘어놓아 작업의 시간 흐름을 재구성한다.

## 권한 / 제약

로컬 데몬에 연결할 수 있는 같은 기기의 사용자·에이전트면 누구나 조회할 수 있는 읽기 전용 경로다. 별도의 변경 권한은 필요 없다. 가져오는 항목 수에는 기본 상한이 있어, 지정하지 않으면 일정 개수까지만 최근순으로 반환된다.

## provenance

CLI 쪽 진입점은 `cli/src/main.rs`의 타임라인 보기 명령으로, 데몬에 타임라인 대시보드 데이터를 요청해 출력한다. 데몬 쪽 타임라인 조회 로직은 `daemon/src/routes/timeline.rs`가 프로젝트별 활동 목록을 시간순으로 모아 돌려주는 형태로 구현되어 있다. 데이터를 소유하고 응답을 만드는 주체는 데몬(component.daemon)이며, 화면 렌더링은 웹 대시보드(component.web-dashboard)가 맡는다.

## 미확정 (OPEN)
- [ ] OPEN: CLI가 호출하는 경로(`/dashboard/timeline`)와 데몬이 실제로 등록한 타임라인 경로(`/projects/{id}/timeline`)의 명칭이 일치하지 않는다. CLI의 타임라인 단축 명령이 데이터 페이로드를 직접 받아오는지, 아니면 웹 뷰만 여는 용도인지(현재 데몬에 `/dashboard/timeline` 라우트가 등록되어 있지 않음) owner 확인 필요.
- [ ] OPEN: 거름 조건으로 지정 가능한 활동 종류의 전체 어휘(태스크/사이클 외에 run 등 포함 여부)와 기본 항목 상한값의 정확한 수치는 코드 정독으로 보강 필요.
