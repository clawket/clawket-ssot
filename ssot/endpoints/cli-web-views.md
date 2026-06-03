---
id: endpoint.cli-web-views
kind: Endpoint
title: clawket timeline / board / wiki / summary
definition: "웹 대시보드의 네 가지 뷰(타임라인·보드·위키·요약)에 해당하는 데이터를 명령줄에서 바로 가져오는 단축 명령들. 각각 해당 뷰의 데이터를 조회해 출력한다."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: component.web-dashboard
    type: reads
    note: "동일한 뷰를 웹 대시보드가 화면으로 렌더링한다 — 이 명령들은 그 뷰의 데이터를 명령줄에서 본다."
  - to: concept.task
    type: reads
    note: "보드(상태별 칸반)·타임라인 뷰가 태스크를 읽어 구성한다."
  - to: concept.cycle
    type: reads
    note: "타임라인 뷰가 사이클의 시간축 흐름을 읽어 구성한다."
  - to: concept.knowledge
    type: reads
    note: "위키 뷰가 지식 항목을 읽어 구성한다."
  - to: concept.plan
    type: reads
    note: "요약 뷰가 active 플랜과 지표 개요를 읽어 구성한다."
governedBy: []
impacts: []
consumedBy:
  - persona.human-operator
  - persona.llm-coding-agent
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
웹 대시보드에는 작업 상태를 보는 화면이 여러 개 있다. 이 네 명령은 그중 네 화면에 대응하는 데이터를 굳이 브라우저를 열지 않고 명령줄에서 바로 받아 보게 해 주는 단축 진입점이다.

- `clawket timeline`: 시간 순서로 태스크·사이클이 흐르는 타임라인 데이터.
- `clawket board`: 태스크 상태(예: 할 일·진행 중·완료)별로 정리한 칸반 보드 데이터.
- `clawket wiki`: 지식 항목 모음(위키) 데이터.
- `clawket summary`: active 플랜과 핵심 지표 개요 데이터.

웹 화면을 띄우지 않고도 상태를 빠르게 확인하려는 운영자나, 화면 대신 구조화된 데이터를 읽으려는 LLM 에이전트가 사용한다.

## 요청 / 응답
- 보내는 정보: 어떤 프로젝트의 뷰를 볼지(생략하면 현재 작업 디렉터리에 연결된 프로젝트를 사용).
- 돌려주는 정보: 각 명령은 해당 뷰를 구성하는 데이터를 그대로 반환한다 — 타임라인은 시간축에 놓인 태스크·사이클 흐름, 보드는 상태별로 묶인 태스크, 위키는 지식 항목 목록, 요약은 active 플랜과 지표 개요. 즉 이 명령들은 브라우저 창을 띄우는 것이 아니라 그 뷰의 내용을 데이터로 받아 출력한다.

## 권한 / 제약
- 모두 읽기 전용 조회다 — 상태를 바꾸지 않는다.
- 로컬 데몬이 살아 있어야 데이터를 가져올 수 있다.
- 프로젝트를 지정하지 않으면 현재 디렉터리 기준 프로젝트로 자동 해석한다.

## provenance
네 명령의 정의·핸들러는 `cli/src/main.rs`에 있다. CLI는 각 뷰에 대응하는 대시보드 데이터 조회를 데몬에 요청하고, 동일한 뷰를 웹 대시보드(`component.web-dashboard`)가 화면으로 그린다.

## 미확정 (OPEN)
- [ ] OPEN: 각 명령이 호출하는 대시보드 뷰 조회 경로가 현재 데몬 라우트(예: 프로젝트별 타임라인/대시보드 라우트)와 정확히 어떻게 매핑되는지 확인 필요(검사 시점 빌드 바이너리와 데몬 소스 간 명령 표면 차이 가능성).
