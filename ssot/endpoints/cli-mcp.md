---
id: endpoint.cli-mcp
kind: Endpoint
title: clawket mcp
definition: "CLI 바이너리에 내장된 MCP stdio 서버를 실행하는 명령으로, Claude Code에 5개의 읽기 전용 지식 도구를 노출한다(데몬이 떠 있어야 동작)."
realizedBy:
  - component.cli
  - component.mcp-server
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - to: integration.mcp
    type: relates-to
    note: 플러그인의 MCP 등록이 이 서버를 Claude Code에 연결한다
  - to: concept.knowledge
    type: reads
  - to: concept.task
    type: reads
  - to: endpoint.mcp-search-knowledge
    type: relates-to
  - to: endpoint.mcp-search-tasks
    type: relates-to
  - to: endpoint.mcp-find-similar-tasks
    type: relates-to
  - to: endpoint.mcp-get-task-context
    type: relates-to
  - to: endpoint.mcp-get-recent-decisions
    type: relates-to
governedBy:
  - invariant.mcp-tools-readonly
  - invariant.mcp-response-cap
impacts: []
consumedBy:
  - component.mcp-launcher
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

코딩 에이전트가 과거 작업 컨텍스트를 의미 기반으로 끌어올 수 있도록, CLI 바이너리 안에 내장된 MCP(stdio) 서버를 띄우는 명령이다. 이 서버가 노출하는 다섯 도구는 모두 읽기 전용이다 — 지식 항목 검색, 태스크 검색, 시드 태스크와 유사한 태스크 찾기, 한 태스크의 전체 컨텍스트(관련 지식·관계·코멘트·이력) 가져오기, 최근 결정성 지식 가져오기. 사람이 직접 이 명령을 치는 일은 거의 없고, Claude Code가 플러그인 설정을 통해 자동으로 이 서버를 stdio로 기동해 도구로 사용한다.

## 요청 / 응답

서버는 표준 입출력으로 MCP 프로토콜을 말한다. 클라이언트(Claude Code)가 도구를 호출하면, 서버는 떠 있는 로컬 데몬의 해당 검색/조회 경로로 요청을 중계하고 결과를 도구 응답으로 돌려준다. 검색 도구는 검색어와 함께 검색 모드(키워드 / 의미 / 둘을 병합한 하이브리드), 반환 개수(상한 30, 기본 10), 그리고 지식 타입이나 특정 플랜 같은 범위 필터를 받는다. 반환되는 각 항목은 사람이 읽을 수 있는 요약 형태이며, 긴 본문은 일정 길이로 잘려 들어간다(일반 항목과 결정성 항목의 잘림 한도가 다르다).

## 권한 / 제약

데몬이 실행 중이어야 한다 — 서버는 시작 시 데몬 포트를 자동 발견해 붙으며, 데몬이 없으면 도구가 응답할 데이터 원천이 없다. 노출 도구는 전부 읽기 전용이라 이 경로로는 어떤 상태도 변경되지 않는다. 응답 크기는 도구별 잘림 한도와 반환 개수 상한으로 묶여, 컨텍스트를 과도하게 채우지 않도록 제한된다.

## provenance

`cli/src/mcp.rs`에서 확인했다. rmcp 기반 stdio 서버 구성, 다섯 읽기 전용 도구의 입력 스키마(검색어·모드·개수·타입/플랜 필터), 개수 상한 30과 일반/결정성 스니펫 잘림 한도, 데몬 자동 발견(`daemon_autostart`) 경유를 추적했다. 명령 진입점은 `cli/src/main.rs`의 `Mcp`이며, 실제 도구 동작은 데몬의 검색·조회 경로에 위임된다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬이 떠 있지 않을 때 자동 기동을 시도하는지, 아니면 오류로 끝나는지(daemon_autostart 분기)와 각 도구의 정확한 응답 필드 묶음은 추가 정독으로 확정 필요.
