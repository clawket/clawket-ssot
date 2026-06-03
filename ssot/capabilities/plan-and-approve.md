---
id: capability.plan-and-approve
kind: Capability
title: 플랜 작성·승인
definition: 에이전트가 작업 의도를 플랜으로 제안·등록하고, 사람이 그것을 승인(approve)해 플랜을 활성(active) 상태로 올림으로써 비로소 그 아래 태스크를 시작할 자격이 생기는 일이다. 승인은 "이 방향으로 작업을 시작해도 좋다"는 명시적 결정 지점이다.
purpose: 작업 의도를 플랜으로 등록하고 승인 게이트를 통과시켜 태스크 시작 자격을 부여한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
  - daemon/src/routes/plans.rs
relatesTo:
  - { to: concept.plan, type: mutates, note: "플랜을 생성하고 draft→active 로 승인 전환" }
  - { to: endpoint.cli-plan, type: calls, note: "plan create / approve 명령으로 수행" }
  - { to: concept.task, type: relates-to, note: "active 플랜이 없으면 태스크 시작 불가" }
  - { to: concept.cycle, type: relates-to, note: "태스크 시작에는 active 플랜과 active 사이클이 함께 필요" }
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

Clawket은 "승인된 의도"인 플랜을 작업의 진실 원천으로 삼는다. 에이전트는 코드베이스를 분석해 어떤 작업을 어떻게 진행할지 대화로 제안하고, 사람이 검토해 승인하면 그 내용을 플랜으로 등록한다. 핵심 가치는 Claude의 Plan Mode 출력 파일이 로컬 폴더에 방치되어 낡아가는 것을 막고, 플랜을 영구 저장소(로컬 DB) 안에 두어 세션을 넘어 살아 있게 만드는 데 있다.

승인은 단순한 저장이 아니라 게이트다. 플랜이 승인되어 활성 상태가 되기 전에는 그 아래 어떤 태스크도 시작할 수 없다. 즉 "방향이 정해지지 않았는데 손부터 대는" 상황을 구조적으로 막는다.

## 행위

- 프로젝트 아래에 플랜을 만든다(작성 시점에는 draft 상태).
- 플랜을 검토 후 승인(approve)하면 active로 전환되어, 그 플랜에 속한 태스크를 시작할 수 있게 된다.
- 작업이 끝난 플랜은 명시적으로 완료(complete) 처리할 수 있다.
- 플랜 내용을 markdown/json/yaml 형식으로 내보내거나 가져올 수도 있다.

## 시스템 흐름

대화에서 플랜을 제안 → 사람이 승인 결정 → 에이전트가 CLI로 플랜 생성, 이어서 승인 호출 → CLI가 데몬 API를 거쳐 SQLite에 플랜을 저장하고 상태를 active로 바꾼다. 이후 태스크 시작 시 가드레일은 "이 태스크가 속한 플랜이 active인가"를 확인한다. Plan Mode를 쓰는 경우, Plan Mode 종료를 가로채는 별도 흐름(plan-sync)이 그 출력을 Clawket 플랜으로 등록하도록 유도한다.

## 어디에 구현되어 있나

명령 진입점은 `cli/src/main.rs`의 plan 서브커맨드, 생성·승인·완료·가져오기/내보내기 처리는 `daemon/src/routes/plans.rs`에 있다. 상태 전이(draft→active→completed)는 데몬이 SQLite에 기록하며, 잘못된 전이는 거부된다.

## 미확정 (OPEN)
- [ ] OPEN: 승인 취소(active→draft 되돌리기)가 허용되는지, 완료된 플랜의 재활성 가능 여부는 코드에서 추가 확인 필요.
