---
id: endpoint.mcp-find-similar-tasks
kind: Endpoint
title: MCP clawket_find_similar_tasks
definition: 시드 태스크와 의미적으로 가까운 과거 태스크를 찾고, 그 본문·코멘트에서 결정과 이슈 패턴을 뽑아 함께 돌려주는 read-only MCP 도구.
realizedBy:
  - component.mcp-server
  - component.daemon
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 시드 태스크의 의미적 이웃(KNN)을 찾거나 자유 쿼리로 유사 태스크를 검색한다
  - to: concept.comment
    type: reads
    note: 유사 태스크의 코멘트·본문에서 결정·이슈 마커를 추출해 extracted 필드로 돌려준다
  - to: endpoint.daemon-search
    type: depends-on
    note: 이웃 검색과 코멘트 조회를 데몬 HTTP API에 위임한다
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

LLM 코딩 에이전트가 "이 작업을 하기 전에, 비슷한 이슈를 해결한 적이 있는지"를 깊이 있게 확인할 때 쓰는 도구다. 단순 검색을 넘어, 닮은 태스크들을 찾은 다음 그 태스크의 본문과 코멘트를 읽어 "어떤 결정을 내렸고 어떤 문제가 있었는지"를 요약해 함께 돌려준다. 검색·조회만 하고 아무것도 바꾸지 않는다.

두 가지 호출 방식이 있다. 기준이 되는 태스크 식별자를 주면 그 태스크의 의미적 이웃(가장 가까운 태스크들)을 찾고, 식별자 없이 자유 검색어만 주면 그 검색어로 의미 검색을 한다. 둘 중 하나는 반드시 줘야 하며, 둘 다 없으면 입력 오류를 돌려준다.

## 요청 / 응답

요청에는 시드 태스크 식별자(있으면 검색어는 무시), 또는 자유 검색어 중 하나를 담는다. 추가로 가져올 개수(1~30개, 기본 5개), 상태 필터, 그리고 결정·이슈 추출을 포함할지 여부(기본 포함)를 지정한다.

응답은 닮은 태스크 목록이며, 각 항목은 태스크 요약(식별자·티켓번호·제목·상태·우선순위·유형·유닛·유사도)에 더해, 추출을 켰을 경우 해당 태스크의 본문·코멘트에서 뽑아낸 결정 목록과 이슈 목록을 담은 추가 묶음(extracted)을 포함한다. 결정·이슈는 한국어·영어로 "결정/확정/Decision/결론/선택" 또는 "이슈/문제/Issue/원인/Root cause" 같은 머리말로 시작하는 줄을 골라 최대 5개씩 모은 것이다.

## 권한 / 제약

읽기 전용이며 데몬이 떠 있어야 동작한다. 시드 식별자도 검색어도 없으면 입력 오류를 돌려준다. 가져올 개수는 최대 30개로 제한된다. 자유 쿼리 방식일 때 상태 필터는 도구 쪽에서 한 번 더 걸러 적용한다. 추출되는 결정·이슈는 정해진 머리말 형식의 줄만 잡아내므로, 자유 서술된 결정은 누락될 수 있다.

## provenance

도구 정의·이웃 검색·코멘트 조회·마커 추출 로직은 모두 `cli/src/mcp.rs`의 `clawket_find_similar_tasks`(및 결정·이슈 정규식 헬퍼)에 있다. 이웃 검색과 코멘트 조회는 데몬 HTTP API(`endpoint.daemon-search`)에 위임하고, CLI 내장 MCP stdio 서버(`component.mcp-server`)가 노출한다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 태스크 유사도(`/tasks/{id}/similar`) 엔드포인트와 코멘트 조회 엔드포인트의 응답 형상을 데몬 라우트 정독으로 확인.
