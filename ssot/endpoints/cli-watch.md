---
id: endpoint.cli-watch
kind: Endpoint
title: clawket watch
definition: "데몬에서 일어나는 엔티티 변경(태스크·플랜·유닛·사이클·지식·실행 기록)을 실시간 스트림으로 받아 터미널에 흘려보내는 CLI 명령으로, 프로젝트·태스크·사이클로 범위를 좁혀 본다."
realizedBy: [component.cli]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.run, type: reads, note: "실행 기록 변경 이벤트를 흘려보낸다" }
  - { to: concept.task, type: reads, note: "태스크 변경 이벤트를 흘려보낸다" }
  - { to: concept.cycle, type: reads, note: "사이클 변경 이벤트를 흘려보낸다" }
governedBy: []
consumesApi: [endpoint.daemon-events]
impacts: []
consumedBy: []
servesPersona: [persona.human-operator, persona.llm-coding-agent]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket watch`는 Clawket 안에서 무언가 바뀔 때마다 그 변화를 곧바로 터미널로 흘려보내는 실시간 관찰 명령이다. 태스크·플랜·유닛·사이클·지식·실행 기록 중 어느 것이 만들어지거나 상태가 바뀌면, 그 사건이 한 줄씩 즉시 출력된다. 사람 운영자가 진행 상황을 옆에 띄워 두고 지켜보거나, 에이전트·스크립트가 변화에 반응해 다음 동작을 트리거할 때 쓴다. 명령은 끄기 전까지(Ctrl-C) 계속 떠 있다.

## 요청 / 응답
- 입력으로 관심 범위를 좁히는 필터를 줄 수 있다 — 특정 프로젝트, 특정 태스크, 특정 사이클에 한정해 그와 관련된 변화만 받는다. 필터를 주지 않으면 데몬이 내보내는 모든 변화를 받는다.
- 출력 형식을 고를 수 있다. 기본은 사람이 읽기 좋은 한 줄 텍스트(사건 식별자·사건 이름·내용)이고, 기계가 파싱하기 좋은 줄 단위 구조화 형식도 선택 가능하다.
- 각 출력 줄은 "무엇이(어떤 엔티티) 어떻게(어떤 사건) 바뀌었는지"를 담는다. 한 사건이 도착할 때마다 즉시 흘려보내므로, 뒤에 연결한 파이프도 사건을 실시간으로 받는다.

## 권한 / 제약
- 이 명령은 읽기 전용 관찰이다 — 아무것도 바꾸지 않는다.
- 변화 사건은 로컬 데몬이 들고 있는 이벤트 스트림에서 나온다. 데몬이 떠 있어야 동작한다.
- 스트림은 명시적으로 끊기 전까지 계속 열려 있다.

## provenance
CLI(`component.cli`)가 명령 표면을 제공하며, 사건은 로컬 데몬의 이벤트 스트림 엔드포인트(`endpoint.daemon-events`)를 구독해 받는다. 명령 정의·필터·출력 형식 처리는 `cli/src/main.rs`의 Watch 분기에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 흘려보내는 사건 종류 전체 목록과 각 사건 내용에 담기는 정보의 정확한 의미는 데몬 이벤트 스트림 명세를 정독해 확정 필요.
