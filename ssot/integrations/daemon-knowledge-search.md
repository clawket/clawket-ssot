---
id: integration.daemon-knowledge-search
kind: Integration
title: 위키 검색 ↔ 데몬 하이브리드 지식 검색 연동
purpose: 사람이 위키에서 검색어를 넣으면 단순 단어 매칭을 넘어 의미가 가까운 지식까지 찾아주되, 검색을 지원하지 않는 옛 데몬에서도 화면이 깨지지 않고 단어 필터로 부드럽게 동작하게 한다.
definition: 웹 대시보드의 위키 검색이 데몬의 지식 검색 엔드포인트를 호출해 키워드 점수(BM25)와 온디바이스 임베딩 기반 의미 점수를 합친 하이브리드 결과를 받는 연동. 검색을 지원하지 않는 옛 데몬이 특정 상태 코드를 돌려주면, 대시보드는 미리 받아둔 지식 목록을 제목·내용·종류 기준으로 거르는 클라이언트 측 단어 필터로 폴백한다.
integratesWith:
  - component.web-dashboard
  - component.daemon
implementedIn:
  - web/src/api.ts
  - web/src/components/WikiView.tsx
impacts:
  - concept.knowledge
  - capability.wiki-knowledge
relatesTo:
  - to: endpoint.daemon-search
    type: backed-by
    note: 하이브리드 검색 결과를 제공하는 데몬 엔드포인트.
  - to: integration.daemon-http
    type: relates-to
    note: 검색도 같은 HTTP 통로 위에서 동작하는 호출의 하나다.
  - to: domain.knowledge-rag
    type: belongs-to
    note: 지식·로컬 RAG 영역의 사람용 검색 표면.
governedBy:
  - invariant.response-shape-backcompat
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드 옮겨적기 금지(식별자는 provenance/근거에만). -->

## 무엇과 연동하나

웹 대시보드의 위키 화면에서 사람이 검색어를 입력하면, 대시보드는 데몬의 **지식 검색 엔드포인트**를 호출한다. 이 검색은 단순한 단어 일치를 넘어선다 — 검색어와 단어가 겹치는 정도(키워드 점수)와, 로컬에서 계산된 임베딩으로 잰 의미적 가까움(의미 점수)을 함께 고려한 **하이브리드 결과**를 돌려준다. 덕분에 정확히 같은 단어를 쓰지 않아도 뜻이 가까운 지식 항목을 찾아낼 수 있다.

핵심 설계는 "버전이 다른 데몬에서도 검색이 늘 동작한다"는 점이다. 의미 검색 기능을 갖추지 못한 옛 데몬을 만나면 검색 엔드포인트가 특정 상태 코드로 "그런 기능 없음"을 알리고, 대시보드는 이를 알아채 **이미 화면에 받아둔 지식 목록을 제목·내용·종류 기준으로 거르는 단순 단어 필터로 폴백한다.** 사용자 입장에선 검색 품질이 달라질 뿐 화면이 멈추거나 비지 않는다.

검색 입력은 매 키 입력마다 곧장 보내지 않고 잠깐 모았다가(디바운스) 한 번에 보내며, 검색 중에는 "검색 중" 표시를, 결과가 없으면 "결과 없음" 표시를 보여준다.

## 구현 위치 (provenance)

- `web/src/api.ts` — 데몬의 지식 검색 호출 함수. 기본 모드는 하이브리드이며, 의미/키워드 모드도 받을 수 있다. 데몬이 검색 엔드포인트를 모르는(옛 버전) 경우에 해당하는 상태 코드를 만나면 결과 없음(null)으로 돌려, 호출부가 클라이언트 측 필터로 폴백하도록 신호한다.
- `web/src/components/WikiView.tsx` — 위키 검색 UI. 입력을 잠깐 모아 한 번에 검색을 보내고, 데몬이 결과를 주면 그 하이브리드 결과를 그대로 화면에 쓰며, 결과 없음(폴백 신호)이면 미리 받아둔 지식 목록을 제목·내용·종류로 거르는 단어 필터로 대체한다.

이 연동을 사용하는 내부 컴포넌트는 웹 대시보드(`component.web-dashboard`)이고, 상대편은 데몬(`component.daemon`)의 지식 검색 엔드포인트(`endpoint.daemon-search`)다.

## 불변식

- 데몬의 검색 응답 형태가 호환되게 유지되어야 하며, 미지원 데몬은 약속된 상태 코드로 부재를 알려야 폴백이 성립한다 — `invariant.response-shape-backcompat`. 이 계약이 깨지면 옛 데몬에서 검색이 오류로 떨어지거나, 신 데몬에서 폴백이 잘못 작동할 수 있다.

## 영향 범위

이 연동은 지식 항목(`concept.knowledge`)을 찾는 사람용 표면이며, 위키 지식 검색 능력(`capability.wiki-knowledge`)을 실현한다. 더 넓게는 지식·로컬 RAG 영역(`domain.knowledge-rag`)의 일부로, 같은 검색 엔진(키워드+의미)을 CLI 검색·MCP 도구와 공유한다. 따라서 검색 엔드포인트의 모드·점수 체계가 바뀌면 위키 화면뿐 아니라 다른 검색 소비처에도 파급된다.

## 미확정 (OPEN)
- [ ] OPEN: 폴백 단어 필터와 하이브리드 결과 사이의 정렬·강조 표시 일관성 기준 확정 필요
- [ ] OPEN: 사용자가 의미/키워드 모드를 직접 고를 수 있게 노출할지 확정 필요
