---
id: capability.browse-wiki-search
kind: Capability
title: 위키 열람·전문/시맨틱 검색
definition: 프로젝트의 작업 디렉터리 파일 트리와 지식 엔트리를 마크다운으로 읽고, 키워드와 의미를 함께 쓰는 하이브리드 검색으로 원하는 문서를 찾는 기능이다.
servesPersona:
  - persona.builder
  - persona.human-operator
realizedBy:
  - component.web
  - component.web-api-client
implementedIn:
  - web/src/components/WikiView.tsx
  - web/src/api.ts
relatesTo:
  - to: concept.knowledge-artifact
    type: reads
    note: 위키에서 다루는 콘텐츠는 task/unit/plan 에 묶인 지식 엔트리(코드상 Artifact 표기)다.
  - to: capability.knowledge-search
    type: relates-to
    note: 데몬의 지식 검색·하이브리드 검색 능력을 화면에서 소비한다.
  - to: domain.knowledge-rag
    type: belongs-to
    note: 위키 열람·검색은 지식·로컬 RAG 영역의 사람용 표면이다.
  - to: screen.web-wiki
    type: belongs-to
    note: 웹 Wiki 화면의 본문이다.
impacts:
  - domain.knowledge-rag
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
사용자는 프로젝트 위키를 한 화면에서 탐색한다. 왼쪽에는 지식 엔트리와 작업 디렉터리(cwd)의 파일이 트리로 놓이고, 항목을 고르면 오른쪽에서 마크다운(GFM 테이블 포함)이나 텍스트로 본문을 읽는다. 검색창에 입력하면 단순 제목 필터를 넘어, 키워드(BM25)와 의미(벡터)를 함께 쓰는 하이브리드 검색으로 관련 지식을 찾는다. 검색은 로컬 데몬이 수행하므로 외부로 데이터가 나가지 않는다.

## 행위
- 지식 엔트리 목록과 cwd 파일 목록을 트리로 표시하고, 트리에서 항목 선택 시 본문을 마크다운/JSON/텍스트 포맷에 맞게 렌더링한다.
- 검색어 입력 시(디바운스 후) 데몬에 하이브리드 모드 검색을 보내 결과 히트를 받는다.
- 데몬 검색이 불가(예: 검색 경로 미지원)하면 클라이언트 측 제목 필터로 폴백한다.
- 검색어가 비면 전체 지식 목록을 그대로 보여준다.
- 검색어에 해당하는 제목/헤더를 하이라이트한다.

## 시스템 흐름
Wiki 화면이 열리면 데몬에서 지식 목록과 위키 파일 목록을 가져와 트리를 만든다. 검색창 입력은 디바운스되어 데몬의 하이브리드 검색 엔드포인트로 전달되고, 반환된 히트가 트리/목록을 대체한다. 데몬 검색이 실패하면 이미 받아 둔 지식을 클라이언트에서 단순 필터링해 끊김 없이 결과를 보여준다. 본문은 마크다운 렌더러로 그린다.

## 어디에 구현되어 있나
웹 앱의 위키 화면 컴포넌트(web/src/components/WikiView.tsx)가 파일·지식 트리, 마크다운 리더, 검색창과 하이라이트를 담당하고, 지식 목록·검색 호출은 데몬 API 래퍼(web/src/api.ts)를 통한다. 웹 Wiki 화면의 본문으로 노출된다.

## 미확정 (OPEN)
- [ ] OPEN: 위키 파일 트리/지식 검색을 서빙하는 데몬 엔드포인트 노드와의 reads/calls 엣지 정렬(web 스코프 전용 노드 미확인 — daemon.wiki/daemon.knowledge 는 desktop 스코프)
