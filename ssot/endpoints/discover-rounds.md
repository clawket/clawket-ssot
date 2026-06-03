---
id: endpoint.discover-rounds
kind: Endpoint
title: 데몬 GET /discover-loop/rounds/{project_id} (라운드별 플랜 목록)
definition: 한 프로젝트의 "Round N" 플랜들을 라운드 번호 순으로 나열하고 각 라운드의 판정 건수를 붙여 단조감소 수렴 추이를 그릴 수 있게 하는 조회 엔드포인트다.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/discover.rs
relatesTo:
  - { to: concept.plan, type: lists, note: "프로젝트의 Round-N 플랜들을 라운드 순으로 나열한다" }
  - { to: concept.discover-round, type: reads, note: "플랜 제목에서 도메인·라운드 번호를 추론해 라운드 단위로 묶는다" }
  - { to: concept.task, type: reads, note: "각 라운드 플랜 산하 태스크의 통과·결함·시나리오오류 건수를 집계한다" }
  - { to: concept.convergence, type: reads, note: "라운드별 결함 건수 추이로 단조감소(수렴) 그래프의 데이터를 제공한다" }
  - { to: endpoint.discover-status, type: complements, note: "status는 한 라운드의 상세, 이 엔드포인트는 전체 라운드의 추이를 준다" }
governedBy: []
impacts: []
consumedBy:
  - endpoint.cli-discover-loop
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

한 프로젝트가 지금까지 거쳐 온 검증 라운드 전체를 라운드 번호 순으로 펼쳐 보여주는 조회 엔드포인트다. 라운드가 거듭될수록 결함이 줄어드는지(수렴 추이)를 한 화면에서 보거나, 단조감소 그래프를 그리는 데 쓴다.

## 요청 / 응답

요청은 경로에 프로젝트 ID를 받는다.

응답은 그 프로젝트의 플랜 중 제목이 "<도메인> Round N" 형식인 것들만 골라, 라운드 번호 오름차순으로 정렬해 목록으로 돌려준다. 각 항목은 플랜 식별·제목·도메인·라운드 번호·플랜 상태, 그리고 그 라운드의 통과·결함·시나리오오류·전체 건수를 함께 담는다. 라운드 형식이 아닌 일반 플랜은 목록에서 빠진다.

## 권한 / 제약

데몬이 살아 있어야 하고 프로젝트 식별이 유효해야 한다. 라운드 인식은 플랜 제목의 "Round N" 패턴에 의존하므로, 라운드 명명 규칙을 따르지 않은 플랜은 추이에 잡히지 않는다.

## provenance

라우트와 핸들러는 daemon/src/routes/discover.rs(/discover-loop/rounds/{project_id}, list_round_plans 핸들러)에 있고, 건수 집계는 같은 파일의 보조 함수를 공유한다. 한 라운드 상세 조회는 endpoint.discover-status, 라운드 개념은 concept.discover-round 에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 라운드 인식이 제목 패턴에만 의존하는 현 방식의 한계(라운드 메타데이터 컬럼 부재)를 구조적으로 보완할지 owner 판단 필요.
