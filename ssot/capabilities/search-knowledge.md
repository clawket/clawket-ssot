---
id: capability.search-knowledge
kind: Capability
title: 위키 검색
definition: Wiki 화면의 검색창에 키워드를 입력하면 지식 항목을 의미·키워드를 함께 보는 하이브리드 방식으로 찾아내고, 동시에 디스크의 마크다운 파일 목록도 같은 검색어로 걸러 트리에 일치하는 항목과 그 상위 폴더만 남긴다.
purpose: 사람이 그동안 쌓인 지식 노트와 프로젝트 안의 문서를 한 검색창에서 빠르게 찾아 읽도록 한다.
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/views/WikiView.tsx
  - desktop/apps/desktop/src/data/api.ts
  - web/src/components/WikiView.tsx
relatesTo:
  - { to: concept.knowledge, type: reads, note: 검색 대상이 되는 지식 항목 }
  - { to: domain.knowledge-rag, type: belongs-to, note: 의미 검색을 제공하는 지식·RAG 영역의 사용자 표면 }
  - { to: endpoint.daemon-search, type: calls, note: 하이브리드 검색을 수행하는 데몬 검색 API 호출 }
  - { to: screen.wiki, type: relates-to, note: 이 기능이 동작하는 화면 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 사용자가 할 수 있는 일

사람이 Wiki 화면 왼쪽 트리 상단의 검색창에 단어를 입력하면, 그동안 쌓인 지식 노트와 프로젝트 안의 마크다운 문서를 한 번에 찾을 수 있다. 검색은 단어가 글자 그대로 들어 있는지(키워드)뿐 아니라 뜻이 비슷한지(의미)도 함께 보는 방식이라, 표현이 조금 달라도 관련된 지식이 걸린다. 검색어를 지우면 다시 전체 트리로 돌아온다.

## 행위

- 검색어를 입력하면 잠깐 멈췄다가(연속 입력이 끝난 뒤) 검색이 시작된다 — 타이핑 도중 매 글자마다 검색하지 않는다.
- 지식 노트는 의미와 키워드를 함께 보는 하이브리드 검색으로 데몬에 질의하고, 결과에 든 항목과 그 위쪽 상위 노트만 트리에 남긴다.
- 디스크의 마크다운 파일 목록은 제목·파일명·경로에 검색어가 들어 있는지로 걸러, 일치하는 파일과 이를 담은 폴더만 남긴다.
- 검색 중에는 "검색 중" 표시가 뜨고, 검색에 실패하면 오류 메시지를, 결과가 없으면 "결과 없음" 안내를 보여준다.

## 시스템 흐름

검색창 입력 → 짧은 지연 후 검색어 확정 → 데몬의 하이브리드 검색 호출(지식) + 화면 안에서의 파일 트리 필터(문서) → 일치한 지식·문서만 남긴 트리를 다시 그림. 검색은 현재 활성 프로젝트 범위로 한정된다.

## 어디에 구현되어 있나

Wiki 화면 컴포넌트가 검색 입력·지연 처리·트리 필터링을 담당하고, 지식 검색 자체는 데몬 클라이언트가 하이브리드 모드로 데몬 검색 API를 호출해 처리한다. 데스크탑 앱과 웹 대시보드가 동일한 화면을 공유한다.

## 미확정 (OPEN)
- [ ] OPEN: 키워드/의미/하이브리드 모드를 사용자가 직접 전환하는 UI는 현재 화면에서 확인되지 않음(하이브리드 고정으로 호출). 모드 선택 노출 여부 확인 필요
