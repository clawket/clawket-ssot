---
id: endpoint.knowledge-search
kind: Endpoint
title: 지식 검색 (데몬 GET /knowledge/search)
definition: 저장된 지식 항목을 키워드·의미·하이브리드 방식으로 검색해 점수가 매겨진 결과 목록을 돌려주는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/knowledge.rs]
relatesTo:
  - { to: concept.knowledge, type: reads, note: "지식 항목을 읽어 검색 대상으로 삼는다." }
  - { to: domain.knowledge-rag, type: belongs-to, note: "로컬 RAG 영역의 검색 표면이다." }
governedBy: []
impacts: [capability.rag-pull, capability.wiki-knowledge]
consumedBy: [endpoint.cli-knowledge, endpoint.daemon-search]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
사용자나 LLM 에이전트가 "예전에 정리해 둔 지식 중 지금 맥락에 맞는 것"을 찾을 때 쓰는 검색 엔드포인트다. 검색어를 보내면 저장된 지식 항목 가운데 관련도가 높은 것을 골라 점수와 함께 돌려준다. 단순 단어 일치(키워드)뿐 아니라 의미가 비슷한 항목까지 찾아내는 의미 검색, 그리고 둘을 섞은 하이브리드 검색을 지원해, 표현이 달라도 같은 주제의 지식을 회수할 수 있다. 세션이 바뀌어 직전 작업 맥락이 사라졌을 때 과거 컨텍스트를 다시 끌어오는 통로로 쓰인다.

## 요청 / 응답
요청에는 검색어, 돌려받을 최대 개수, 검색 방식(키워드만 / 의미만 / 하이브리드), 그리고 특정 프로젝트로 범위를 좁히는 선택 조건을 담는다. 검색 방식을 지정하지 않으면 하이브리드가 기본이며, 개수는 일정 범위(최소 1개, 최대 100개) 안으로 자동 보정된다.

응답으로는 검색 결과 목록과 함께, 실제로 돌려준 개수·요청한 한도·결과가 한도에 걸려 잘렸는지 여부가 온다. 결과의 각 항목은 지식 본문에 더해 "왜 이게 걸렸는지"를 가늠하게 하는 세 가지 점수(키워드 일치 강도, 의미 유사도, 둘을 가중 합산한 종합 점수)를 담는다. 하이브리드에서는 의미 유사도에 더 큰 비중을, 키워드 일치에 보조 비중을 두어 의미가 우선되되 정확한 단어 매칭도 반영되도록 순위를 매긴다. 프로젝트 범위가 지정되면 그 프로젝트에 속한 지식만 남기고, 어느 플랜·유닛·태스크에도 매이지 않은 지식(사용자 단위 지식)은 모든 프로젝트에서 함께 노출된다.

## 권한 / 제약
로컬 데몬에 붙은 클라이언트(CLI·웹·MCP)가 호출하며 읽기 전용이라 데이터를 바꾸지 않는다. 의미·하이브리드 검색은 검색어를 임베딩으로 변환할 수 있을 때만 동작하고, 임베딩에 실패하면 키워드 검색으로 자동 강등(fallback)되어 빈손 대신 부분 결과라도 돌려준다. 프로젝트 범위 필터가 걸리면 걸러진 뒤에도 한도를 채우도록 내부적으로 더 넉넉히 후보를 가져온다.

## provenance
구현은 데몬 라우트 daemon/src/routes/knowledge.rs 의 `/knowledge/search` 핸들러에 있으며, 키워드 검색·벡터 검색·임베딩 변환을 조합해 결과를 합산·정렬한다. 제공 주체는 로컬 데몬(component.daemon)이고, 의미 검색의 점수 근거가 되는 벡터는 임베딩 엔진이 만든 것이다. 코드 구조가 사라져도, "키워드·의미·하이브리드 세 모드, 하이브리드는 의미 우선 가중, 임베딩 실패 시 키워드 강등, 프로젝트 범위 필터와 사용자 단위 지식의 무조건 노출"이라는 정책만으로 재구현이 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 하이브리드 가중치(의미 0.7 / 키워드 0.3)가 설계 고정값인지 향후 조정 대상인지 확인 필요 — 판정은 owner.
