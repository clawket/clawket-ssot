---
id: component.web-dashboard
kind: SystemComponent
title: 웹 대시보드 (React 19 + Vite)
definition: 데몬의 HTTP API와 SSE 이벤트 버스에 붙어 Summary·Board·Backlog·Timeline·Wiki 다섯 뷰와 사이드바 Plans 트리를 렌더링하고 드래그앤드롭 상태 변경을 제공하는 React SPA 번들.
purpose: 사람 운영자가 LLM 에이전트와 동일한 작업 상태를 눈으로 보고 직접 관리하는 표면을 제공하기 위해 존재한다. 칸반 보드 드래그앤드롭으로 태스크 상태를 옮기고, 백로그를 사이클에 배정하며, 에이전트별 스윔레인 타임라인으로 동시 작업을 시각화하고, 위키에서 지식을 버전 관리·검색한다. CLI가 LLM의 표면이라면 웹은 사람의 표면이며, 둘 다 같은 상태를 본다.
realizedBy:
  - capability.web-review
  - capability.wiki-knowledge
  - persona.human-operator
implementedIn:
  - web/src
  - web/dist
  - web/package.json
dependsOn:
  - component.daemon
consumesApi:
  - endpoint.daemon-events
  - endpoint.daemon-search
  - endpoint.daemon-health
providesApi: []
integratesWith:
  - integration.github-releases
impacts:
  - persona.human-operator
  - domain.work-management
  - domain.knowledge-rag
relatesTo:
  - to: component.daemon
    type: depends-on
    note: 모든 데이터·실시간 갱신을 데몬 HTTP API + SSE에서 가져온다
  - to: component.desktop-app
    type: relates-to
    note: 미출시 데스크탑 앱이 동일 SPA를 렌더링한다
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 책임

사람 운영자(`persona.human-operator`)를 위한 시각적 작업관리 표면이다. 데몬이 떠 있을 때 로컬 주소(기본 포트 19400)로 접근하며, 다섯 개의 최상위 뷰와 항상 보이는 Plans 트리(왼쪽 사이드바)로 구성된다.

- Summary: 진행도·활성 에이전트·유닛 진행을 한눈에 보는 프로젝트 개요.
- Board: 칸반 보드. 카드를 끌어 상태를 바꾼다.
- Backlog: 사이클별로 묶인 백로그. 끌어서 사이클에 배정한다.
- Timeline: 에이전트별 스윔레인(실행 바)과 활동 스트림 탭으로 동시 작업을 시각화한다.
- Wiki: 파일 트리 탐색, 지식 CRUD와 버전 이력, 키워드+의미 검색, 마크다운 표 렌더링.

Plans 트리는 인라인 편집·일괄 동작·체크박스 선택을 사이드바에서 제공해, 탭 뒤가 아니라 모든 뷰에서 접근된다.

## 경계와 의존

웹은 정적 SPA 번들로, 자체 백엔드를 갖지 않는다. 모든 데이터 읽기·쓰기와 실시간 갱신은 데몬(`component.daemon`)의 HTTP API와 SSE 이벤트 버스를 통한다. 따라서 데몬에 의존하며, 데몬이 떠 있지 않으면 대시보드는 데이터를 받지 못한다. 번들 자체는 GitHub Releases 타르볼로 배포되어 설치 게이트가 풀어 놓는다(`integration.github-releases`).

미출시 데스크탑 앱(`component.desktop-app`)은 이 웹과 동일한 SPA를 렌더링하도록 설계돼 있으나, 컴포넌트 버전 핀이 비어 있어 현재는 설치되지 않는다.

## 통신 패턴

웹 → 데몬 방향의 HTTP 요청으로 상태를 읽고 쓰며, 데몬의 SSE 스트림(`endpoint.daemon-events`)을 구독해 task/cycle/run 변경을 실시간 반영한다. 위키 검색은 데몬의 키워드·의미·하이브리드 검색 엔드포인트(`endpoint.daemon-search`)를 호출한다. 드래그앤드롭으로 인한 상태 변경(칸반 상태 이동, 백로그→사이클 배정)은 해당 갱신 API 호출로 데몬에 반영된다.

## 하위 서브패키지 (책임 단위)

- 뷰 계층(Summary/Board/Backlog/Timeline/Wiki): 다섯 최상위 화면.
- Plans 사이드바 트리: 플랜·유닛·태스크 계층의 인라인 편집·일괄 동작.
- 데이터 훅(API 클라이언트 + SSE 구독): 데몬과의 통신·실시간 갱신.

## 미확정 (OPEN)
- [ ] OPEN: 웹이 호출하는 쓰기 API(상태 변경·DnD 배정)의 정확한 엔드포인트 매핑은 데몬 라우트 노드(미작성분)에 위임. 현재 인벤토리에 개별 쓰기 엔드포인트 노드가 없어 OPEN으로 둔다.
