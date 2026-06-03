---
id: capability.session-restore
kind: Capability
title: 세션 복원(태스크 풀컨텍스트 풀)
purpose: "한 태스크의 본문·실행기록·코멘트·관련 지식·관계·이력을 한 번에 묶어 가져와, 끊긴 세션을 이어받는 LLM 프롬프트의 시드로 쓴다."
servesPersona:
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.mcp-server
  - component.daemon
implementedIn:
  - cli/src/main.rs
  - cli/src/mcp.rs
relatesTo:
  - { to: domain.knowledge-rag, type: relates-to, note: "세션 간 의미 컨텍스트 복원 도메인" }
  - { to: concept.task, type: relates-to, note: "맥락을 모아 올 중심 엔티티" }
  - { to: concept.knowledge, type: relates-to, note: "태스크에 첨부된 지식 항목을 함께 반환" }
  - { to: concept.run, type: relates-to, note: "태스크의 실행 기록을 함께 반환" }
  - { to: concept.comment, type: relates-to, note: "태스크 코멘트를 함께 반환" }
  - { to: endpoint.cli-rag-aliases, type: calls, note: "get-task-context 최상위 별칭 명령" }
  - { to: endpoint.mcp-get-task-context, type: calls, note: "동일 기능을 노출하는 read-only MCP 도구" }
impacts:
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 사용자가 할 수 있는 일

LLM 에이전트는 대화 세션이 바뀌면 직전까지의 맥락을 잃는다. 이 기능은 특정 태스크 하나를 지목하면 그 주변 정보를 한 묶음으로 돌려준다 — 태스크 본문과 상태, 그동안의 실행 기록(run), 사람·에이전트가 남긴 코멘트, 첨부된 지식 항목, 다른 태스크와의 관계(차단·관련·중복), 그리고 활동 이력. 이 한 페이로드를 그대로 새 세션의 프롬프트에 넣으면 "이 티켓이 무슨 배경으로 생겼고 지금 어디까지 왔는지"를 즉시 복원할 수 있다.

## 행위

- **태스크 지목**: 태스크 ID(TASK-… 또는 티켓번호) 하나를 입력한다.
- **묶음 조회**: 기본으로 지식·관계를 포함해 가져오고, 코멘트·이력은 명시적으로 추가 요청할 수 있다(MCP 도구 기준 include 옵션).
- **단일 페이로드 반환**: 태스크 메타(제목/상태/우선순위/유형/소속 유닛·사이클/시작·완료 시각)와 함께 요청한 부가 정보가 하나의 JSON으로 합쳐져 나온다 — LLM 프롬프트에 바로 파이프할 수 있는 형태.

## 시스템 흐름

진입점은 두 가지다. CLI에서는 `endpoint.cli-rag-aliases`의 get-task-context 별칭으로, Claude Code 안에서는 read-only MCP 도구 `endpoint.mcp-get-task-context`로 부른다. 둘 다 데몬의 태스크·지식·관계·코멘트·활동 조회 API를 병렬로 호출해 결과를 하나로 합친다. 데이터를 바꾸지 않는 순수 읽기 동작이다.

## 어디에 구현되어 있나

CLI 별칭 선언은 `cli/src/main.rs`에, MCP 도구 본체는 `cli/src/mcp.rs`에 있다. MCP 쪽은 태스크 조회 후 지식·관계·코멘트·이력 경로를 동시에 가져와(`tokio::join!`) 하나의 응답 맵으로 조립한다 — 기본 포함은 지식·관계, 코멘트·이력은 include 지정 시에만 합쳐진다.

## 미확정 (OPEN)
- [ ] OPEN: CLI get-task-context와 MCP 도구 간 반환 형상 차이(필드/스니펫 길이) 여부 확인 필요
