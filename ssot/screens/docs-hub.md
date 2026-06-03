---
id: screen.docs-hub
kind: Screen
title: 문서 허브 화면(Docs Hub)
definition: /docs 경로의 문서 레이아웃 겸 개요 화면. 좌측 사이드바 내비게이션(Overview + 4개 섹션)과 우측 본문(Outlet) 구조를 가지며, 허브 자체에서는 Clawket 한 단락 소개, 4개 섹션 카드(Getting Started·Workflow·CLI Reference·Architecture), 그리고 정본(source-of-truth) 레포 링크를 보여 준다.
purpose: 문서를 찾아온 사람이 설치·워크플로우·CLI·아키텍처 네 갈래 중 필요한 곳으로 이동하고, 더 깊은 정본(플러그인 레포와 4개 컴포넌트 레포)의 위치를 알도록 안내한다.
servesPersona: [persona.human-operator, persona.llm-coding-agent]
realizedBy: [component.landing-site]
implementedIn: [landing/src/pages/Docs.tsx]
consumesApi: []
relatesTo:
  - { to: screen.docs-getting-started, type: leads-to, note: "사이드바/카드의 Getting Started 진입점" }
  - { to: screen.docs-workflow, type: leads-to, note: "사이드바/카드의 Workflow 진입점" }
  - { to: screen.docs-cli, type: leads-to, note: "사이드바/카드의 CLI Reference 진입점" }
  - { to: screen.docs-architecture, type: leads-to, note: "사이드바/카드의 Architecture 진입점" }
  - { to: screen.home, type: relates-to, note: "헤더 ← clawket 링크로 랜딩 메인으로 복귀" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

문서 영역의 진입점이자 공통 레이아웃이다. /docs 루트에서는 개요(허브) 본문을 보여 주고, 하위 경로(/docs/getting-started 등)에서는 같은 레이아웃 안의 본문 자리에 해당 문서 페이지를 끼워 넣는다. 방문자는 좌측 내비게이션으로 네 섹션 사이를 오가고, 허브 본문의 카드로 각 섹션이 무엇을 다루는지 미리 파악한 뒤 들어간다.

## UI 요소 / 입력 필드

- 상단 헤더: 랜딩 메인으로 돌아가는 "← clawket" 링크, "Docs" 라벨, 언어 전환기.
- 좌측 사이드바 내비게이션: Overview(허브 자체)와 4개 섹션 링크(Getting Started, Workflow, CLI Reference, Architecture). 현재 보고 있는 섹션이 강조 표시된다.
- 우측 본문: /docs 루트면 허브 개요를, 하위 경로면 해당 문서 페이지를 렌더한다.
- 허브 개요 본문: Clawket 한 단락 소개(로컬 우선·태스크에 변경을 고정·근거 없이는 done 거부·SQLite에 플랜/유닛/태스크/지식 저장·FTS5+sqlite-vec 하이브리드 검색), 네 섹션 카드(각 카드에 제목과 한 줄 요약), 그리고 "Source of truth" 박스(플러그인 레포 clawket/clawket 와 cli·daemon·web·desktop 4개 형제 레포로 가는 링크).

입력 필드는 없고 상호작용은 링크 이동과 언어 전환뿐이다.

## 표시 데이터 / 호출 API

표시되는 문구는 내비게이션 정의와 본문에 하드코딩된 영문 텍스트다(이 문서 영역은 다국어 사전을 쓰지 않고 영문 고정). 외부 링크는 GitHub 레포들을 가리킨다. 데몬·데이터베이스를 호출하지 않으므로 consumesApi 가 없다.

## 상태 / 엣지케이스

- /docs 와 /docs/ 두 경로 모두 허브 개요로 처리한다.
- 현재 경로에 따라 사이드바 항목과 Overview 항목의 활성 강조가 갈린다.
- 하위 페이지로 전환할 때 빈 head 깜빡임을 막기 위해 허브 레이아웃이 임시 메타데이터를 채워 두고, 하위 페이지가 자기 메타데이터로 덮어쓴다.
- 정적 콘텐츠라 로딩/에러 상태는 없다.

## 미확정 (OPEN)
- 없음
