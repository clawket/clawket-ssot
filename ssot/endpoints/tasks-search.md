---
id: endpoint.tasks-search
kind: Endpoint
title: 데몬 GET /tasks/search
definition: 키워드·의미·하이브리드 모드로 태스크를 검색하는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon, component.embedding-engine]
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - { to: concept.task, type: reads, note: "검색어에 맞는 태스크를 점수순으로 반환한다" }
  - { to: endpoint.daemon-search, type: backed-by, note: "태스크 키워드(FTS5)·의미(벡터) 검색 인프라를 공유한다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-task, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

검색어로 태스크를 찾는 기능이다. 단순 키워드 매칭뿐 아니라 의미가 비슷한 태스크까지 찾아낼 수 있어, 사람이 "예전에 비슷한 일을 했던 태스크"를 떠올리거나 LLM 에이전트가 과거 작업 맥락을 끌어올릴 때(RAG 풀) 바탕이 된다.

## 요청 / 응답

요청에는 검색어와 함께 검색 방식을 지정한다. 방식은 네 가지다 — 키워드 매칭, 의미 기반(임베딩 유사도), 둘을 합친 하이브리드, 그리고 태스크의 실행 계약(의도·프롬프트·성공기준) 본문만 뒤지는 봉투 전용 검색이다. 봉투 전용 모드는 태스크 본문에는 한 번도 안 나왔지만 그 작업의 계약에는 언급된 주제를 찾는 용도라, 일반 본문으로 내려가 재검색하지 않는다. 가져올 개수도 줄 수 있으며 기본 20건·최대 100건으로 제한된다. 응답은 검색 점수 순으로 정렬된 태스크 목록이며, 의미 검색이 동작한 항목에는 유사도 거리 같은 점수 정보가 함께 붙는다.

## 권한 / 제약

로컬 데몬에 한정된 읽기 전용 검색이다. 방식을 지정하지 않으면 키워드 매칭이 기본값이다. 의미·하이브리드 검색은 온디바이스 임베딩 엔진이 검색어를 벡터로 만들 수 있을 때만 의미 검색이 적용되고, 임베딩이 실패하면 키워드 검색으로 안전하게 떨어진다. 결과 개수는 1~100건 범위로 강제 보정된다.

## provenance

`daemon/src/routes/tasks.rs`의 `/tasks/search` 라우트가 모드에 따라 태스크 저장소의 키워드 검색(FTS5)·벡터 검색·봉투 전용 검색을 호출하고, 하이브리드는 두 결과를 중복 없이 병합한다. 제공 주체는 clawketd 데몬(component.daemon)이고 의미 검색은 온디바이스 임베딩 엔진(component.embedding-engine)에 의존한다. CLI task 명령과 웹 대시보드가 소비한다.

## 미확정 (OPEN)
- [ ] OPEN: 하이브리드 결과의 최종 랭킹 기준(키워드 우선 병합 외 가중치 적용 여부)을 저장소 계층에서 확인해 보강할 것.
