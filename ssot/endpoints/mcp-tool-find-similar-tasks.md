---
id: endpoint.mcp-tool-find-similar-tasks
kind: Endpoint
title: MCP 도구 clawket_find_similar_tasks (구현 정의)
definition: 시드 태스크의 KNN 이웃 또는 자유 쿼리로 유사 태스크를 찾고, 태스크 본문·코멘트에서 결정·이슈 마커를 추출해 함께 돌려주는 read-only MCP 도구.
realizedBy: [component.mcp-server, component.cli]
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - { to: concept.task, type: reads, note: "시드 태스크의 의미적 이웃을 찾아 반환한다" }
  - { to: concept.comment, type: reads, note: "태스크 본문·코멘트에서 결정/이슈 마커를 추출해 extracted에 담는다" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

LLM 코딩 에이전트가 "이 작업 전에 비슷한 이슈를 해결한 적 있나"를 확인할 때 부르는 read-only MCP 도구다. 단순 검색(clawket_search_tasks)보다 한 걸음 더 나아가, 비슷한 과거 태스크를 찾는 데 그치지 않고 그 태스크의 본문과 코멘트에서 결정·이슈 흔적을 골라내 함께 돌려줘 "그때 무슨 결정을 했고 무슨 문제를 만났나"까지 한 번에 회수하게 한다.

## 요청 / 응답

요청은 두 방식 중 하나로 시작한다 — 기준 태스크ID를 주면 그 태스크의 의미적 최근접 이웃(KNN)을 찾고, 검색어를 주면 자유 쿼리 의미 검색을 한다. 둘 다 비면 입력 오류로 거절한다. 추가로 결과 상한(기본 5건, 내부 상한으로 캡), 상태 필터, 그리고 마커 추출 포함 여부(기본 포함)를 줄 수 있다. 응답으로는 유사 태스크 요약 목록이 돌아오고, 마커 추출이 켜져 있으면 각 항목에 그 태스크 본문+코멘트를 합쳐 정규식으로 뽑아낸 결정 목록과 이슈 목록(각 최대 5개)이 extracted 필드로 덧붙는다.

## 권한 / 제약

데몬이 살아 있어야 동작한다. read-only — 태스크나 코멘트를 바꾸지 않는다. 기준 태스크ID와 검색어 중 적어도 하나는 반드시 있어야 한다. 태스크ID 경로는 데몬의 유사 태스크 KNN 엔드포인트를, 검색어 경로는 의미 검색 엔드포인트를 쓰며, 상태 필터는 검색어 경로에서 도구 측이 한 번 더 걸러 적용한다. 마커 추출은 항목마다 코멘트를 추가로 조회하므로 결과 수가 많을수록 데몬 호출이 늘어난다.

## provenance

도구 정의·요청 처리·마커 추출 로직은 cli/src/mcp.rs(215-310행)에 있다. 사람이 읽는 도구 요약은 endpoint.mcp-find-similar-tasks 에 별도로 정리되어 있다. KNN 검색은 데몬의 유사 태스크 엔드포인트와 검색 HTTP API(endpoint.daemon-search)에 위임되며, CLI 바이너리에 내장된 MCP 서버(component.mcp-server)가 노출하는 지식 도구군의 하나다.

## 미확정 (OPEN)
- [ ] OPEN: 결정/이슈 마커 추출에 쓰는 정규식(decision_regex/issue_regex)이 인식하는 표기 규칙과 다국어 대응 범위는 해당 헬퍼 정독으로 확인.
