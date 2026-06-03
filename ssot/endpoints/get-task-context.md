---
id: endpoint.get-task-context
kind: Endpoint
title: 태스크 전체 컨텍스트 가져오기 (get-task-context)
definition: "clawket get-task-context — 태스크 본문·실행 기록·코멘트·연결된 지식을 하나의 JSON 묶음으로 돌려준다. LLM 세션 복원용으로 프롬프트에 그대로 흘려넣기 좋게 만든 것."
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.task, type: reads, note: "대상 태스크 본문을 읽는다." }
  - { to: concept.run, type: reads, note: "태스크의 실행 기록을 함께 묶는다." }
  - { to: concept.comment, type: reads, note: "태스크의 코멘트를 함께 묶는다." }
  - { to: concept.knowledge-entry, type: reads, note: "태스크에 연결된 지식 엔트리를 함께 묶는다." }
  - { to: endpoint.mcp-get-task-context, type: relates-to, note: "동일 기능을 LLM 에이전트에 노출하는 MCP 도구 짝." }
governedBy: []
impacts: []
consumedBy: [capability.session-restore]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

`clawket get-task-context` 는 하나의 태스크에 얽힌 작업 맥락을 한 번에 끌어오는 명령이다. 태스크 본문은 물론, 그 태스크의 실행 기록(run), 주고받은 코멘트, 연결된 지식 엔트리까지 하나의 JSON 묶음으로 돌려준다. 여러 번 따로 조회하지 않고도 "이 태스크의 모든 것"을 단일 호출로 받게 하는 것이 핵심이다.

용도가 분명하다 — LLM 세션 복원이다. 새 세션을 시작할 때 이 묶음을 그대로 프롬프트에 흘려넣으면, 이전에 무슨 작업이었고 어떤 실행과 논의·결정이 있었는지를 모델이 한 번에 회복한다. 세션 간 작업 컨텍스트를 잇는 다리 역할을 한다.

## 요청 / 응답

- 보내는 정보: 태스크 식별자 하나.
- 돌려주는 정보: 해당 태스크의 본문 + 실행 기록 목록 + 코멘트 목록 + 연결된 지식 엔트리 목록을 한 묶음으로. LLM 프롬프트에 바로 붙일 수 있는 형태를 의도한다.

## 권한 / 제약

로컬 단일 사용자 환경 전제. 읽기 전용 조회이므로 상태를 바꾸지 않는다. 묶음 크기는 해당 태스크에 쌓인 실행/코멘트/지식 양에 비례한다.

## provenance

CLI 명령 정의·핸들러는 `cli/src/main.rs` 에 있다(`GetTaskContext { id }`). CLI 는 데몬의 `/tasks/{id}/context` 경로를 읽기(GET) 호출하도록 작성되어 있다. 같은 컨텍스트 묶음은 MCP 도구(get-task-context)로도 LLM 에이전트에 노출된다.

## 미확정 (OPEN)
- [ ] OPEN: CLI 는 데몬 `/tasks/{id}/context` 를 호출하지만 현재 데몬 라우터(`daemon/src/routes/tasks.rs` 의 `router()`)에 해당 경로 핸들러가 등록되어 있지 않다. 컨텍스트 묶음을 조립하는 구현 위치(또는 미구현 여부)를 확인해 realizedBy/implementedIn 을 확정할 것.
- [ ] OPEN: 묶음에 실제로 포함되는 항목 범위(예: 모든 run vs 최신 N건, 지식 엔트리 선별 기준)와 정렬·상한을 데몬 구현에서 확정할 것.
