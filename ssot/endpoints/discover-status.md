---
id: endpoint.discover-status
kind: Endpoint
title: 데몬 GET /discover-loop/status (라운드 수렴 상태 조회)
definition: 활성 라운드 플랜의 통과·결함·시나리오오류 건수를 집계해 직전 라운드와 비교하고 회귀 여부·수렴 여부를 돌려주는 조회 엔드포인트다.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/discover.rs
relatesTo:
  - { to: concept.convergence, type: reads, note: "결함=0·시나리오오류=0 여부와 직전 라운드 대비 회귀를 판정해 보고한다" }
  - { to: concept.plan, type: reads, note: "라운드 플랜과 직전 라운드 플랜을 찾아 제목에서 라운드 번호를 추론한다" }
  - { to: concept.task, type: reads, note: "플랜 산하 태스크의 판정 상태를 집계해 건수를 센다" }
  - { to: concept.discover-round, type: reads, note: "Round N 단위로 결과를 묶어 단조감소 비교의 한 점을 만든다" }
  - { to: concept.knowledge, type: mutates, note: "도메인별 수렴 감사 로그 지식에 라운드별 결정 한 줄을 누적 기록한다" }
  - { to: endpoint.discover-converged, type: complements, note: "이 조회는 한 라운드 스냅샷, converged는 2라운드 연속 최종 판정을 담당한다" }
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

지금 라운드가 얼마나 수렴에 가까운지를 한눈에 보여주는 조회 엔드포인트다. 검증 루프 스킬이 한 라운드의 전사가 끝난 뒤 이 엔드포인트를 호출해 "이번 라운드에 결함이 몇 개고, 직전 라운드보다 나아졌는지 나빠졌는지"를 확인하고 다음 행동(수렴 처리/다음 라운드/회귀 경보)을 정한다.

## 요청 / 응답

요청은 대상 플랜을 직접 지정하거나(플랜 ID), 프로젝트만 줘서 그 프로젝트의 활성 플랜을 자동으로 잡게 할 수 있다.

응답은 그 라운드의 통과·결함·시나리오오류·전체 건수와 함께, 라운드 번호(플랜 제목 "<도메인> Round N"에서 뽑는다), 그리고 두 가지 판정을 돌려준다 — 결함과 시나리오오류가 둘 다 0이면 "이 라운드는 수렴"이라는 표시, 그리고 결함이 0보다 크면서 직전 라운드보다 늘었으면 "회귀 감지" 표시다. 직전 라운드가 있으면 그 라운드의 건수 요약도 함께 담아 비교의 근거를 보여준다. 통과 건수 집계는 라운드 결과 컬럼(qa_status)과 작업 상태 컬럼(status) 중 어느 한쪽이라도 결함을 가리키면 결함으로 세는 방식이라, 수동 편집으로 두 신호가 어긋난 경우(상태만 차단으로 바꾸고 라운드 결과는 안 채운 경우)도 놓치지 않는다.

조회와 동시에 도메인별 "수렴 감사 로그" 지식에 이번 라운드 한 줄(라운드 번호·결함·시나리오오류·통과·결정값)을 누적해, 라운드가 거듭될수록 결함이 줄어드는 추이가 한 문서에 쌓이게 한다(같은 줄이 이미 마지막에 있으면 중복 기록하지 않는다).

## 권한 / 제약

데몬이 살아 있어야 한다. 플랜 ID도 프로젝트 ID도 주지 않으면 거절하고, 프로젝트만 준 경우 활성 플랜이 없으면 못 찾았다고 응답한다. 이 엔드포인트는 한 라운드의 스냅샷만 본다 — "2라운드 연속 0이면 최종 수렴"이라는 안전 마진 판정은 별도의 converged 엔드포인트가 맡는다.

## provenance

라우트와 핸들러는 daemon/src/routes/discover.rs(/discover-loop/status, convergence_status 핸들러)에 있고, 건수 집계 SQL과 직전 라운드 탐색 보조 함수가 같은 파일에 있다. 최종 수렴 판정은 endpoint.discover-converged, 수렴 개념 정의는 concept.convergence·invariant.convergence-two-rounds 에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 회귀 감지(결함이 직전보다 증가)가 경보에 그치는지 후속 동작을 강제하는지, 수렴 감사 로그 지식의 보존·열람 흐름을 owner 확인.
