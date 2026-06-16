---
id: endpoint.mcp-get-task-context
kind: Endpoint
title: MCP clawket_get_task_context
definition: 한 태스크와 그 주변 맥락(관련 지식·관계·코멘트·활동 이력)을 한 번의 호출로 묶어 돌려주는 read-only MCP 도구.
realizedBy:
  - component.mcp-server
  - component.daemon
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 대상 태스크의 핵심 속성과 주변 맥락을 한 페이로드로 묶는다
  - to: concept.knowledge
    type: reads
    note: 태스크에 첨부된 지식 항목을 스니펫으로 포함한다
  - to: concept.comment
    type: reads
    note: 요청 시 태스크 코멘트를 포함한다
  - to: concept.audit-log
    type: reads
    note: 요청 시 태스크의 활동 이력을 포함한다
  - to: endpoint.daemon-search
    type: depends-on
    note: 태스크·지식·관계·코멘트·이력 조회를 데몬 HTTP API에 위임한다
  - to: domain.knowledge-rag
    type: belongs-to
governedBy:
  - invariant.mcp-tools-readonly
  - invariant.mcp-response-cap
impacts:
  - capability.rag-pull
consumedBy:
  - persona.llm-coding-agent
  - integration.mcp
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

LLM 코딩 에이전트가 "이 티켓이 무슨 배경에서 만들어졌는지"를 한눈에 파악하려 할 때 쓰는 도구다. 태스크 하나를 지목하면, 그 태스크의 기본 정보와 함께 관련 지식·태스크 간 관계·코멘트·활동 이력을 한 번에 묶어 돌려준다. 여러 도구를 따로 부를 필요 없이 맥락을 한 묶음으로 받는 것이 목적이다. 조회만 하고 아무것도 바꾸지 않는다.

## 요청 / 응답

요청에는 태스크 식별자(ULID 또는 티켓번호 CK-N)를 담고, 어떤 섹션을 포함할지 선택한다. 포함 가능한 섹션은 관련 지식, 태스크 관계, 코멘트, 활동 이력 네 가지이며, 지정하지 않으면 기본으로 지식과 관계만 포함한다. 코멘트와 이력은 무거울 수 있어 명시적으로 요청해야 붙는다.

응답은 항상 태스크 본체(식별자·티켓번호·제목·상태·우선순위·유형·소속 유닛·소속 사이클·본문·생성/시작/완료 시각)를 담고, 요청한 섹션을 덧붙인다. 지식은 각 항목의 제목·타입·스니펫으로, 관계는 막는 것·막힌 것·관련됨·중복 네 갈래로 묶여, 코멘트는 작성자·시각·내용으로, 이력은 활동 기록 목록으로 돌아온다. 결과가 너무 커지면 잘림 안내와 함께 앞부분만 돌려준다.

## 권한 / 제약

읽기 전용이며 데몬이 떠 있어야 동작한다. 데몬에 닿지 못하면 구조화된 오류를 돌려준다. 활동 이력은 최근 50건까지로 제한된다. 각 섹션 조회는 병렬로 이루어지며, 특정 섹션 조회가 실패해도 빈 값으로 채워 전체 응답은 유지한다.

## provenance

도구 정의와 섹션별 조회·병합 로직은 `cli/src/mcp.rs`의 `clawket_get_task_context`에 있다. 태스크·지식·관계·코멘트·활동 조회는 데몬 HTTP API(`endpoint.daemon-search` 및 데몬의 태스크/지식/활동 라우트)에 위임하고, CLI 내장 MCP stdio 서버(`component.mcp-server`)가 노출한다.

## 미확정 (OPEN)
- [ ] OPEN: 활동 이력(`/activity`)이 감사 로그(`concept.audit-log`)와 동일 출처인지, 별도 활동 스트림인지 데몬 라우트 정독으로 확인(현재 reads 로 추정 연결).
- [ ] OPEN: 태스크 관계 4갈래(blocks/blocked_by/relates_to/duplicates)에 대응하는 별도 concept 노드가 인벤토리에 있는지 확인.
