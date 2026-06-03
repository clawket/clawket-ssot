---
id: endpoint.mcp
kind: Endpoint
title: clawket mcp
definition: "Claude Code가 stdio로 붙는 MCP 서버를 실행해 5개 read-only 지식 도구를 노출하며, 시작 시 데몬이 떠 있지 않으면 자동 기동한다."
realizedBy: [component.cli, component.mcp-server]
implementedIn: [cli/src/main.rs, cli/src/mcp.rs]
relatesTo:
  - { to: integration.mcp-claude-code, type: relates-to, note: "이 명령이 Claude Code에 등록되는 MCP 서버 본체" }
  - { to: endpoint.mcp-search-knowledge, type: relates-to, note: "노출하는 read-only 도구 중 하나" }
  - { to: endpoint.mcp-search-tasks, type: relates-to, note: "노출하는 read-only 도구 중 하나" }
  - { to: endpoint.mcp-find-similar-tasks, type: relates-to, note: "노출하는 read-only 도구 중 하나" }
  - { to: endpoint.mcp-get-task-context, type: relates-to, note: "노출하는 read-only 도구 중 하나" }
  - { to: endpoint.mcp-get-recent-decisions, type: relates-to, note: "노출하는 read-only 도구 중 하나" }
governedBy: []
impacts: []
consumedBy: [integration.mcp-claude-code]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
LLM 코딩 에이전트(Claude Code)가 Clawket의 누적 컨텍스트를 의미 검색으로 끌어올 수 있게 해 주는 다리다. 이 명령을 실행하면 표준입출력(stdio) 위에서 MCP 프로토콜을 말하는 서버가 떠서, 다섯 개의 읽기 전용 지식 도구를 도구 목록으로 광고한다. 에이전트는 이 도구들을 호출해 과거 결정·지식 항목·관련 태스크를 세션 간에 다시 불러올 수 있다. 사용자가 직접 손으로 실행하기보다는, 플러그인이 Claude Code에 등록해 둔 진입점이 이 명령을 stdio 서버로 기동하는 형태로 쓰인다.

서버가 노출하는 도구는 전부 조회 전용이며, 검색(지식·태스크), 유사 태스크 찾기(코멘트에서 결정·이슈 추출 포함), 태스크 풀컨텍스트 조회, 최근 결정 조회다.

## 요청 / 응답
- 시작 시: 별도 인자 없이 stdio 서버로 동작한다. 실제 데이터 조회 요청은 에이전트가 개별 도구 호출(tool_use)로 보내며, 각 도구가 받는 입력(검색어, 범위 등)과 돌려주는 결과(지식·태스크 묶음)의 의미는 각 도구 엔드포인트 문서에 정의된다.
- 결과 크기: 응답이 지나치게 커지면 중간에서 잘려 의미가 깨지지 않도록, 일정 한도에서 깔끔히 "잘렸다"는 표시와 함께 잘라 돌려준다.

## 권한 / 제약
모든 도구가 읽기 전용이라 이 경로로는 어떤 상태도 변경되지 않는다(변경은 CLI 쓰기 명령·훅을 통해서만). 서버는 데몬의 HTTP API에 의존하므로, 시작할 때 데몬이 떠 있지 않으면 먼저 자동으로 기동을 시도한다. 자동 기동이 실패해도 서버 자체는 죽지 않고, 이후 개별 도구 호출이 연결 오류를 구조화된 오류로 돌려준다(중단/패닉이 아니라 호출 단위 실패). 로그는 프로토콜 채널(표준출력)을 오염시키지 않도록 표준오류로만 나간다.

## provenance
명령 정의와 설명은 CLI 진입점(`cli/src/main.rs`)의 mcp 하위명령에, 서버 기동·데몬 자동 기동·응답 크기 한도 등 동작은 `cli/src/mcp.rs`에 있다. 제공 주체는 CLI 바이너리에 내장된 MCP 서버 컴포넌트이며, Claude Code 쪽 등록은 별도 MCP 연동이 담당한다.

## 미확정 (OPEN)
- [ ] OPEN: 각 도구가 주고받는 데이터의 정확한 의미는 해당 도구 엔드포인트 문서로 위임 — 여기서는 5종 노출 사실만 확정.
