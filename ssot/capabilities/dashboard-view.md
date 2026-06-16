---
id: capability.dashboard-view
kind: Capability
title: 대시보드·뷰 페이로드 조회(CLI)
purpose: "활성 프로젝트의 작업 요약과 타임라인·보드·위키·서머리 뷰 데이터를 데몬에서 받아, '지금 내가 어디에 있는지'를 세션 시작과 검토 시점에 한눈에 본다."
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: domain.work-management, type: relates-to, note: "요약·뷰가 비추는 작업 관리 도메인" }
  - { to: concept.project, type: relates-to, note: "요약의 기준이 되는 활성 프로젝트" }
  - { to: concept.plan, type: relates-to, note: "요약에 담기는 활성 플랜" }
  - { to: concept.cycle, type: relates-to, note: "요약에 담기는 활성 사이클" }
  - { to: concept.task, type: relates-to, note: "요약에 담기는 in_progress 태스크" }
  - { to: endpoint.cli-dashboard, type: calls, note: "활성 프로젝트 작업 요약 렌더 명령" }
  - { to: endpoint.cli-web-views, type: calls, note: "timeline/board/wiki/summary 뷰 단축 명령" }
impacts:
  - concept.project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 사용자가 할 수 있는 일

작업을 시작하거나 검토할 때 가장 먼저 필요한 것은 "지금 활성 프로젝트에서 무엇이 진행 중인가"라는 한 장의 그림이다. 이 기능은 활성 프로젝트의 작업 요약(활성 플랜·유닛·사이클·진행 중인 태스크)을 만들어 보여주고, 더 자세히 보려는 사람을 위해 타임라인·보드·위키·서머리 네 가지 뷰의 데이터를 꺼내 준다. Claude Code의 세션 시작 훅이 이 요약을 컨텍스트 시드로 쓰기 때문에, 사람뿐 아니라 에이전트도 매 세션 처음에 같은 그림에서 출발한다.

## 행위

- **작업 요약 렌더**: 작업 디렉터리로부터 활성 프로젝트를 해석한 뒤, 활성 플랜·유닛·사이클과 진행 중(in_progress) 태스크를 요약한다. active|next|all로 보이는 범위를 거를 수 있다.
- **뷰 데이터 조회**: 타임라인(시간순 태스크·사이클 리본), 보드(상태별 칸반), 위키(지식 항목), 서머리(활성 플랜+KPI)의 페이로드를 프로젝트 단위로 가져온다.

## 시스템 흐름

대시보드 요약은 `endpoint.cli-dashboard`로 들어가 데몬의 대시보드 요약 데이터를 받아 렌더한다. 네 뷰는 `endpoint.cli-web-views`(timeline/board/wiki/summary)로 들어가 각각 데몬의 대시보드 뷰 API(`/dashboard/timeline`·`/board`·`/wiki`·`/summary`)를 프로젝트 ID로 질의한다. 모두 데몬이 집계해 둔 읽기 데이터를 받아 출력하는 흐름이다.

## 어디에 구현되어 있나

CLI 명령 정의와 라우팅은 `cli/src/main.rs`에 있다. dashboard 명령은 작업 디렉터리 해석과 show 필터를 받아 요약을 그리고, Timeline/Board/Wiki/Summary 명령은 각각 데몬의 대시보드 뷰 엔드포인트를 프로젝트 쿼리로 호출한다.

## 미확정 (OPEN)
- [ ] OPEN: 각 뷰 페이로드가 CLI에서 텍스트로 렌더되는지, 웹 대시보드를 여는 단축인지의 동작 구분 정밀 확인 필요
