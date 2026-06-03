---
id: endpoint.summary
kind: Endpoint
title: 요약(Summary) 대시보드 페이로드 조회
definition: "GET /dashboard/summary — 활성 플랜과 핵심 지표(KPI)를 한 장에 모은 요약 화면용 데이터를 데몬에서 가져온다."
realizedBy:
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.plan
    type: reads
    note: 현재 active 상태인 플랜을 골라 요약 화면의 머리에 올린다.
  - to: concept.task
    type: reads
    note: 태스크 상태 분포를 집계해 진행률 등 KPI 지표로 환산한다.
  - to: concept.cycle
    type: reads
    note: 현재 활성 사이클을 함께 표시해 지금 어느 타임박스 안에 있는지 보여준다.
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

요약 뷰는 프로젝트의 현재 상태를 한 장으로 압축해 보여주는 개요 화면이다. 지금 활성화된 플랜이 무엇인지, 그 아래 작업이 얼마나 진행됐는지(완료/진행/할 일 비율 같은 KPI), 지금 돌아가는 사이클은 무엇인지를 묶어 보여준다. 사람 운영자가 대시보드에 들어왔을 때 가장 먼저 보는 "지금 상황 한눈에" 성격의 화면이며, 이 노드는 그 요약을 채우기 위한 데이터를 데몬에서 받아오는 조회 경로를 가리킨다.

조회 단위는 하나의 프로젝트이며, 프로젝트를 지정하지 않으면 현재 작업 디렉터리에 연결된 프로젝트를 대상으로 한다.

## 요청 / 응답

요청에는 어느 프로젝트의 요약을 볼지(생략 시 현재 디렉터리 프로젝트)를 보낸다.

응답은 활성 플랜의 식별 정보와, 그 플랜·프로젝트에 대해 집계한 핵심 지표들이다. 대표적으로 태스크 상태별 개수에서 환산한 진행률, 현재 활성 사이클, 그리고 처리가 필요한 항목(예: 답변 대기 중인 질문, 활성 플랜이 둘 이상일 때의 경고 같은 운영 신호)이 포함된다. 요약 화면은 이 데이터를 받아 상단 개요와 지표 카드들을 그린다.

## 권한 / 제약

로컬 데몬에 연결 가능한 같은 기기의 사용자·에이전트가 조회할 수 있는 읽기 전용 경로다. 활성 플랜이 정해져 있지 않으면 요약은 "활성 플랜 없음" 상태로 비게 되며, 활성 플랜이 권장 수(보통 1개)를 넘으면 응답에 경고성 신호가 함께 실린다.

## provenance

CLI 쪽 진입점은 cli/src/main.rs 의 요약 보기 명령으로, 데몬에 요약 대시보드 데이터를 요청해 출력한다. 활성 플랜 선택·유닛/태스크 진행률 집계·활성 사이클·대기 질문·활성 플랜 과다 경고 같은 요약성 신호를 모으는 로직은 데몬의 대시보드 핸들러(daemon/src/routes/dashboard.rs)에 이미 존재하므로, 요약 페이로드는 이 집계 결과를 기반으로 한다. 데이터를 소유하는 주체는 데몬(component.daemon)이며, 렌더링은 웹 대시보드(component.web-dashboard)가 맡는다.

## 미확정 (OPEN)
- [ ] OPEN: CLI가 호출하는 /dashboard/summary 경로에 대응하는 전용 데몬 라우트가 현재 등록되어 있지 않다(데몬은 /dashboard 단일 집계 경로만 가짐). CLI의 요약 단축 명령이 데이터를 직접 받는지 웹 뷰만 여는지, 그리고 요약 페이로드가 기존 /dashboard 집계를 재사용하는지 별도 핸들러인지 owner 확인 필요.
- [ ] OPEN: 요약에 노출되는 KPI 항목의 정확한 목록과 계산식(진행률 정의, 어떤 경고 신호까지 포함하는지)은 코드 정독으로 보강 필요.
