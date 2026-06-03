---
id: invariant.mcp-response-cap
kind: Invariant
title: MCP 도구 응답은 100KB에서 안내와 함께 잘린다
definition: MCP 도구가 돌려주는 응답은 100KB를 넘지 않도록 잘리고, 잘릴 때는 "잘렸음"을 알리는 안내 문구가 붙는다 — Claude Code가 50KB 부근에서 응답을 조용히 잘라 버리는 것을 피하기 위해서다.
governs:
  - component.mcp-server
  - capability.rag-pull
  - endpoint.cli-mcp
crossesBoundary: false
implementedIn:
  - cli/src/mcp.rs
decidedBy:
  - decision.daemon-single-source
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 제약

MCP 도구(LLM 에이전트가 지식·태스크를 읽어 가는 통로)가 돌려주는 한 번의 응답은 일정 상한(약 100KB)을 넘지 못한다. 응답이 그보다 커지면 시스템이 앞부분만 남기고 잘라낸 뒤, 그 끝에 "응답이 상한을 초과해 잘렸으니 더 구체적인 필터를 쓰거나 개수를 줄이라"는 안내 문구를 붙여 돌려준다. 즉, 큰 결과는 "조용히 사라지는" 것이 아니라 "잘렸다는 사실이 보이는" 형태로 전달된다.

## 깨지면 무슨 일이 일어나나

Claude Code는 도구 결과가 일정 크기(50KB 부근)를 넘으면 사용자도 모델도 모르게 중간에서 잘라 버린다. 이 절단은 JSON 같은 구조화된 응답의 한가운데를 끊어, 모델이 깨진 데이터를 온전한 데이터로 착각하게 만든다. 만약 MCP가 자기 응답 크기를 스스로 통제하지 않으면 바로 이 무성(無聲) 절단에 노출되어, 에이전트가 잘린 줄도 모르고 부분 결과를 전체로 오인한 채 잘못된 판단을 내린다. 그래서 그 한계선보다 더 여유 있는 지점(약 100KB)에서 미리, 그리고 명시적으로 자르고 안내를 붙여 "결과가 더 있다"는 신호를 반드시 남긴다.

## 코드에서 어떻게 강제되나

MCP 성공 응답을 만드는 경로(`cli/src/mcp.rs`)는 직렬화한 응답 텍스트의 크기를 상한과 비교한다. 상한 이내면 그대로 돌려주고, 상한을 넘으면 앞부분만 남긴 뒤 "상한 초과로 잘림 — 더 구체적인 필터를 쓰거나 개수를 줄이라"는 안내 문구를 덧붙여 성공 응답으로 반환한다. 코드 주석은 이 상한이 Claude Code의 무성 절단(50KB 부근)을 피하려는 의도이며, 깔끔한 "잘림" 메시지를 주기 위해 100KB에서 자른다고 명시한다.

## 미확정 (OPEN)
- [ ] OPEN: 100KB라는 상한 값과 절단 정책을 확정한 명시적 결정 기록(decidedBy)이 별도 decision 노드로 존재하는지 미확인 — 현재는 데몬 단일 진실 결정에 임시 연결.
