---
id: endpoint.find-similar
kind: Endpoint
title: 유사 태스크 찾기 (find-similar)
definition: "clawket find-similar — 질의 텍스트와 의미적으로 비슷한 태스크를 벡터 검색으로 찾는다. task search --mode semantic 의 최상위 단축 명령."
realizedBy: [component.cli, component.daemon, component.embedding-engine]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.tasks-search, type: backed-by, note: "task 의미검색(mode=semantic)의 단축 명령 — 같은 벡터 검색 기능을 가리킨다." }
  - { to: concept.task, type: reads, note: "검색 대상은 태스크(제목+본문)." }
  - { to: endpoint.mcp-find-similar-tasks, type: relates-to, note: "동일 기능을 LLM 에이전트에 노출하는 MCP 도구 짝." }
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

`clawket find-similar` 는 자유 텍스트 질의를 주면, 그 질의와 의미적으로 가까운 태스크들을 찾아주는 명령이다. 단순 키워드 일치가 아니라 벡터(임베딩) 기반 유사도 검색이라, 표현이 달라도 뜻이 비슷한 태스크를 끌어온다. 검색 대상은 태스크의 제목과 본문이다.

기능상 `clawket task search --mode semantic` 과 같은 의미검색을 가리키는 최상위 단축 명령이다. "이거랑 비슷한 일 전에 있었나?" 를 빠르게 확인하는 용도이며, 사람뿐 아니라 LLM 에이전트가 과거 유사 작업을 참고할 때도 쓴다(같은 기능이 MCP 도구로도 노출된다).

옵션:

- **최대 결과 수(limit)**: 기본 10건.
- **프로젝트 한정(project)**: 특정 프로젝트 범위로 좁힐 수 있다.

## 요청 / 응답

- 보내는 정보: 질의 텍스트, 최대 결과 수, (선택) 프로젝트 범위.
- 돌려주는 정보: 질의와 유사한 태스크 목록. 각 항목은 태스크 식별 정보와 함께 유사도 점수를 가진다.

유사도가 너무 낮은 결과가 결과 집합을 오염시키지 않도록, 일정 유사도 미만(코사인 유사도 약 0.3 미만)은 잘라내는 하한이 적용된다.

## 권한 / 제약

로컬 단일 사용자 환경 전제. 의미검색이므로 태스크 본문에 대한 임베딩이 준비되어 있어야 결과가 나온다. 유사도 하한 컷오프(약 0.3) 때문에, 충분히 가까운 태스크가 없으면 결과가 비어 돌아올 수 있다.

## provenance

CLI 명령 정의·핸들러는 `cli/src/main.rs` 에 있다(`FindSimilar { query, limit, project }`). CLI 는 데몬의 `/tasks/similar?q=...` 경로를 읽기(GET) 호출하도록 작성되어 있다. 유사도 하한(코사인 0.3) 정책은 데몬의 태스크 라우트(`daemon/src/routes/tasks.rs` 의 `similar` 핸들러, RAG-SIM-004)에서 확인된다. 벡터 검색 자체는 임베딩 엔진/스토어가 뒷받침한다.

## 미확정 (OPEN)
- [ ] OPEN: CLI 는 질의형 경로 `/tasks/similar?q=` 를 호출하지만, 현재 데몬 라우터에 등록된 유사 검색 경로는 태스크 ID 기반 `/tasks/{id}/similar` 와 의미검색 `/tasks/search?mode=semantic` 이다. CLI 가 호출하는 질의형 경로의 실제 라우트 등록 여부(또는 search 로의 매핑 의도)를 확인해 backed-by 대상을 확정할 것.
- [ ] OPEN: 결과 항목이 담는 정보(제목/본문 발췌/상태/점수 등)의 정확한 구성을 데몬 응답에서 확정할 것.
