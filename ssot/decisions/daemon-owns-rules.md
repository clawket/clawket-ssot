---
id: decision.daemon-owns-rules
kind: Decision
title: 판정 규칙은 데몬이 단독 소유하고 MCP·웹은 얇은 소비자로 둔다
purpose: "분해 제안·검증 같은 판정 로직을 어디에 둘지 — 표면(CLI·MCP·웹)마다 각자 구현할지, 한곳에 모아 표면은 호출만 하게 할지"
definition: "태스크 분해 제안·검증 등 '무엇이 옳은가'를 정하는 판정 규칙을 데몬 한곳에 모으고, MCP 도구와 웹 화면은 그 규칙을 직접 들고 있지 않은 채 데몬을 호출하기만 하는 얇은 소비자(thin client)로 둔다."
relatesTo:
  - to: component.daemon
    type: relates-to
    note: 분해·검증 등 판정 규칙을 단독으로 소유하는 주체
  - to: concept.decomposition
    type: relates-to
    note: 데몬이 규칙을 소유하는 대표 사례 — 분해 제안 로직이 한곳에 산다
  - to: integration.mcp
    type: relates-to
    note: 규칙을 들고 있지 않고 데몬을 호출만 하는 얇은 소비자 표면
  - to: component.web-dashboard
    type: relates-to
    note: 같은 분해 엔드포인트를 호출하는 또 다른 얇은 소비자 표면
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

<!-- Decision은 append-only. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 supersedes로 잇는다. -->

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다. (스냅샷 원칙 — 히스토리는 여기와 git에만)

## 맥락 (Context)

Clawket의 같은 기능(예: 태스크를 하위 태스크로 쪼개는 분해 제안, 검증 결과 판정)은 여러 표면에서 노출된다. LLM 에이전트는 MCP 도구로, 사람은 웹 화면의 제안 패널로 같은 분해 제안을 본다. 만약 각 표면이 "어떻게 쪼개는 게 옳은가"라는 판정 규칙을 제 코드 안에 따로 들고 있으면, 표면마다 규칙이 조금씩 달라지며(rule divergence) MCP에서 본 제안과 웹에서 본 제안이 어긋난다. 시간이 지나면 어느 것이 진짜 규칙인지조차 모호해진다.

## 결정 (Decision)

판정 규칙을 데몬 한곳에 모은다. 분해 제안 로직은 데몬 내부 모듈에 살고, MCP 도구와 웹 제안 패널은 둘 다 같은 데몬 엔드포인트(태스크 분해 요청)를 호출해 결과만 받아 보여준다. 규칙은 표면이 아니라 데몬이 소유하며, 표면은 규칙의 사본을 두지 않는다.

## 근거와 결과 (Consequences)

- **표면 간 규칙 일관성**: 규칙이 한곳에 있으므로 MCP·웹·CLI가 보는 판정이 항상 동일하다. single source of truth 원칙을 판정 로직에 적용한 결과다.
- **변경의 단일 지점**: 규칙을 고칠 때 데몬만 바꾸면 모든 표면에 즉시 반영된다. 표면별로 흩어진 동일 로직을 일일이 동기화하는 비용·실수가 사라진다.
- **얇은 소비자**: MCP·웹은 규칙을 모르는 채 호출만 하므로 표면 구현이 가벼워지고, 새 표면을 붙일 때도 규칙을 재구현하지 않는다.
- **확장 방향**: 같은 패턴이 검증 판정에도 적용된다 — 데몬이 규칙을 소유하고, 검증 표면은 데몬 판정을 소비한다.

provenance: 분해 제안 모듈 주석이 명시 — CLI MCP 도구와 웹 제안 패널이 같은 분해 엔드포인트를 통해 이 모듈을 소비하므로 "표면 간 규칙이 발산할 수 없다", "daemon owns the rules, MCP and web are thin clients" (`daemon/src/decomposition/mod.rs`).
