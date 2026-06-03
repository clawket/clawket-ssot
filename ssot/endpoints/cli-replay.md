---
id: endpoint.cli-replay
kind: Endpoint
title: clawket replay
definition: "한 태스크의 실행 기록을 시간 순서대로 다시 풀어 보여주는 CLI 명령으로, 작업이 끝난 뒤 무슨 일이 어떤 순서로 일어났는지 사후 점검할 때 쓴다."
realizedBy: [component.cli]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.run, type: reads, note: "태스크에 쌓인 실행 기록을 순서대로 읽어 재생한다" }
  - { to: concept.task, type: reads, note: "지정한 태스크의 실행 이력을 대상으로 한다" }
governedBy: []
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
`clawket replay`는 한 태스크를 둘러싸고 그동안 쌓인 실행 기록을 시간 순서대로 다시 펼쳐 보여주는 명령이다. 태스크는 여러 번 실행될 수 있고 매 실행마다 시작·종료 시각, 어떤 에이전트가 했는지, 결과는 어땠는지, 어떤 메모가 남았는지가 기록된다. 이 명령은 그 기록들을 순서대로 나열해, 작업이 끝난 뒤 "무엇이 어떤 순서로 일어났는가"를 되짚는 사후 점검(post-mortem) 용도로 쓴다.

## 요청 / 응답
- 입력으로 재생할 대상 태스크를 지정한다. 최근 몇 개의 실행까지 펼칠지 개수를 제한할 수 있다(지정하지 않으면 기본 개수).
- 출력은 그 태스크의 실행 기록들을 시간 순서로 나열한 것으로, 각 실행의 시작·종료, 수행 주체, 결과, 메모를 담는다.

## 권한 / 제약
- 읽기 전용 점검이다 — 기록을 다시 보여줄 뿐 아무것도 바꾸지 않으며, 실행을 실제로 재실행하지도 않는다.
- 대상 태스크에 실행 기록이 남아 있어야 의미가 있다. 실행 기록은 보통 작업 진행 중 훅이 자동으로 만든다.

## provenance
CLI(`component.cli`)가 명령 표면을 제공하며, 실제 기록은 로컬 데몬이 보관한 태스크별 실행 이력에서 가져온다. 명령 정의와 개수 제한 처리는 `cli/src/main.rs`의 Replay 분기에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 실행 기록 한 건에 담기는 결과·메모 필드의 정확한 의미와, 같은 태스크의 여러 실행을 비교하는 방식은 데몬 실행 이력 라우트를 정독해 확정 필요.
