---
id: endpoint.discover-start
kind: Endpoint
title: 검증 라운드 시작 (라운드 R 플랜·사이클·QA유닛 자동 생성)
definition: POST /discover-loop/start — 한 번의 요청으로 라운드 R의 승인된 플랜, 병렬 QA 유닛들, 활성 사이클을 묶어 만들어 검증 라운드가 곧장 태스크를 받을 수 있는 상태로 세팅한다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/discover.rs]
relatesTo:
  - { to: concept.plan, type: mutates, note: "라운드 플랜을 draft로 만든 뒤 곧바로 승인해 active 상태로 만든다" }
  - { to: concept.unit, type: mutates, note: "QA 영역별로 병렬 모드 유닛을 만든다" }
  - { to: concept.cycle, type: mutates, note: "첫 유닛에 앵커된 사이클을 만들고 즉시 활성화한다" }
  - { to: domain.pdd-verification, type: realizes, note: "검증·수렴 루프의 라운드 진입 절차를 구현한다" }
governedBy: []
impacts: [concept.plan, concept.cycle, concept.unit]
consumedBy: [endpoint.cli-discover-loop]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

검증(PDD) 루프의 한 라운드를 "시작 가능한 상태"로 한 번에 세팅하는 명령이다. 검증 도메인(예: "대시보드", "Chess 학습")과 라운드 번호, 그리고 검증할 영역들의 목록을 받으면, 그 라운드에 필요한 작업 컨테이너 일체 — 승인된 플랜 한 개, 영역별 QA 유닛 여러 개, 활성 사이클 한 개 — 를 자동으로 만들어 준다. 검증 루프를 도는 스킬 에이전트나 운영자가 라운드를 새로 열 때 호출한다. 이 한 번의 호출로 "플랜 만들고 → 승인하고 → 유닛 나누고 → 사이클 만들고 → 활성화한다"는 다섯 단계를 손으로 밟지 않아도 된다.

## 요청 / 응답

보내는 정보:
- 라운드가 속할 프로젝트
- 사람이 읽는 도메인 이름 — 만들어지는 플랜·유닛·사이클의 제목에 들어간다(플랜 제목은 "<도메인> Round <번호>" 규칙).
- 라운드 번호(1 이상이어야 한다)
- QA 영역 목록 — 영역 하나가 유닛 하나가 되며, 모두 병렬 실행 모드로 만들어진다. 최소 한 개는 있어야 한다.
- (선택) 플랜 설명. 생략하면 수렴 조건(결함 0, 시나리오오류 0, 마지막 2라운드 연속 0)이 적힌 설명이 자동으로 채워져, 나중에 이 플랜을 검증하는 하위 에이전트가 문서를 따로 찾지 않고도 합격 기준을 인라인으로 본다.

돌려주는 정보: 방금 만들어진 플랜(이미 활성), 만들어진 유닛 목록, 활성화된 사이클.

## 권한 / 제약

- 영역 목록이 비어 있으면 거절한다("영역이 최소 하나 필요").
- 라운드 번호가 0이면 거절한다("1 이상이어야 함").
- 사이클은 첫 번째 유닛에 구조적으로 앵커되지만, 같은 사이클에 다른 유닛의 태스크도 함께 묶일 수 있다(교차 유닛 사이클 허용 — 검증 루프 롤백 설계). 즉 사이클의 소속 유닛은 그저 닻일 뿐, 태스크 배정 범위를 가두지 않는다.
- 이미 이 프로젝트에 활성 플랜이 있으면 작업을 막지는 않되 경고 신호를 남긴다. 권장은 활성 플랜 1개 이하이며 전환기에는 2개까지 용인한다 — 그 이상은 작업이 너무 잘게 쪼개졌다는 냄새로 본다.
- 사이클은 만들자마자 활성화되므로 라운드 태스크가 곧바로 시작 가능하다.

## provenance

데몬의 `daemon/src/routes/discover.rs`(start 핸들러, POST `/discover-loop/start`)가 제공한다. CLI `clawket discover-loop start` 가 이 엔드포인트의 사용자 표면이다. 플랜·유닛·사이클의 생성·승인·활성화 정책 자체는 각 엔티티 개념 노드의 진실을 따르며, 여기서는 "라운드 단위로 그것들을 한 번에 세팅한다"는 오케스트레이션 의미만 보존한다.

## 미확정 (OPEN)
- [ ] OPEN: 활성 플랜이 권장치를 넘었을 때 남기는 경고가 호출자에게 어떻게 노출/소비되는지(이벤트 구독 주체) 확인 필요.
