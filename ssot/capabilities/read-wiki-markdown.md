---
id: capability.read-wiki-markdown
kind: Capability
title: 위키 마크다운 읽기
definition: Wiki 화면에서 지식 항목이나 디스크의 마크다운 파일을 선택하면, 그 본문을 표·취소선·체크리스트 같은 확장 문법(GFM)까지 갖춘 서식 있는 문서로 렌더링해 읽기 좋게 보여준다.
purpose: 사람이 마크다운 원문이 아니라 제목·목록·표가 정리된 읽기 좋은 형태로 지식과 문서를 본다.
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/views/WikiView.tsx
  - web/src/components/WikiView.tsx
relatesTo:
  - { to: concept.knowledge, type: reads, note: 본문을 렌더링해 보여주는 지식 항목 }
  - { to: screen.wiki, type: relates-to, note: 마크다운이 표시되는 화면 }
  - { to: capability.search-knowledge, type: relates-to, note: 검색으로 찾은 항목을 선택해 본문을 읽는 흐름으로 이어짐 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 사용자가 할 수 있는 일

사람이 Wiki 화면의 트리에서 지식 노트나 프로젝트 문서 파일을 고르면, 오른쪽 본문 영역에 그 내용이 서식 있는 문서로 표시된다. 제목·목록·표·코드 블록·링크 같은 마크다운 요소가 제대로 꾸며져 나오고, 표나 체크리스트 같은 GitHub 스타일 확장 문법도 함께 렌더링되어, 원문 텍스트가 아니라 읽기 좋은 형태로 지식을 볼 수 있다.

## 행위

- 지식 항목을 선택하면 그 항목의 종류(예: wiki·decision·note)와 제목·작성 시각을 헤더로 보여주고, 본문을 서식 있는 문서로 렌더링한다.
- 디스크 파일을 선택하면 데몬에서 해당 파일 내용을 읽어와 파일 경로·이름·수정 시각과 함께 본문을 렌더링한다.
- 선택한 것이 없으면 "읽을 항목을 고르라"는 안내를, 파일을 불러오는 중이면 로딩 표시를, 읽기에 실패하면 오류 메시지를 보여준다.

## 시스템 흐름

트리에서 항목 선택 → 지식이면 화면이 이미 들고 있는 내용을, 파일이면 데몬에서 파일 내용을 받아와 → 마크다운 렌더러가 서식 있는 문서로 그림. 파일 읽기 중에 다른 항목을 빠르게 다시 고르면 가장 최근 요청의 결과만 반영한다.

## 어디에 구현되어 있나

Wiki 화면 컴포넌트가 선택 상태를 관리하고, 지식 본문은 화면이 가진 데이터에서 바로, 디스크 파일 본문은 데몬 호출로 가져와 마크다운 렌더러로 그린다. 데스크탑 앱과 웹 대시보드가 동일 화면을 공유한다.

## 미확정 (OPEN)
- [ ] OPEN: 렌더링되는 마크다운 안의 링크 클릭(내부 노트 간 이동 등) 동작 여부 확인 필요
