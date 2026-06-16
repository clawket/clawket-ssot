---
id: capability.command-palette
kind: Capability
title: 명령 팔레트
definition: Cmd/Ctrl+K(또는 상단바 버튼)로 화면 위에 검색창을 띄워, 뷰·계획·단위·작업을 한 곳에서 검색하고 화살표·엔터로 골라 해당 화면으로 이동하거나 항목을 선택한다.
purpose: 사람이 마우스로 메뉴를 헤매지 않고 키보드만으로 원하는 화면·계획·단위·작업에 즉시 닿게 한다.
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/App.tsx
  - desktop/apps/desktop/src/hooks/useGlobalShortcuts.ts
  - desktop/packages/ui/src/components/CommandSurface/CommandSurface.tsx
  - web/src/components/CommandPalette.tsx
  - web/src/hooks/useGlobalShortcuts.ts
relatesTo:
  - { to: concept.plan, type: reads, note: 팔레트에서 검색·이동 대상이 되는 계획 }
  - { to: concept.unit, type: reads, note: 검색·이동 대상이 되는 단위 }
  - { to: concept.task, type: reads, note: 티켓 번호·제목으로 검색·이동하는 작업 }
  - { to: screen.board, type: relates-to, note: 뷰 항목 선택으로 전환되는 대시보드 화면들 }
  - { to: capability.keyboard-shortcuts, type: relates-to, note: 팔레트를 여는 단축키를 제공하는 기능 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 사용자가 할 수 있는 일

사람이 Cmd/Ctrl+K를 누르거나 상단바의 ⌘K 버튼을 누르면 화면 한가운데에 검색창이 뜬다. 여기에 단어를 입력하면 화면(뷰)·계획·단위·작업이 한 목록으로 묶여 검색되고, 위아래 화살표로 항목을 옮기며 엔터로 고르면 그 화면으로 이동하거나 해당 계획·단위·작업이 선택된다. 키보드만으로 앱 전체를 빠르게 오갈 수 있는 단일 진입점이다.

## 행위

- 단축키나 버튼으로 팔레트를 연다. 열리면 입력창에 자동으로 커서가 맞춰진다.
- 검색어가 없을 때는 뷰 전체와 계획·단위·작업의 앞부분만 그룹별로 보여주어 목록이 한눈에 들어오게 하고, 검색어를 넣으면 제목 또는 작업 티켓 번호에 그 단어가 든 항목만 추린다.
- 화살표 위/아래로 선택을 옮기고(목록 끝에서 순환), 엔터나 클릭으로 활성화한다. Esc나 바깥 클릭으로 닫는다.
- 항목을 고르면 뷰는 화면 전환, 작업·계획·단위는 해당 항목 선택으로 이어지고, 팔레트는 닫히며 검색어가 비워진다.

## 시스템 흐름

단축키/버튼 → 팔레트 열림 → 입력에 따라 뷰·계획·단위·작업을 모은 목록을 즉석에서 추림 → 화살표·엔터로 선택 → 선택 종류에 따라 화면 전환 또는 항목 선택 → 팔레트 닫힘. 목록은 화면이 이미 들고 있는 데이터에서 만들어지며 별도 서버 호출 없이 즉시 걸러진다.

## 어디에 구현되어 있나

상단 앱 컴포넌트가 팔레트 열림 상태와 검색어, 그리고 뷰·계획·단위·작업을 묶은 목록 구성과 선택 처리를 맡는다. 검색창 표면(입력·화살표/엔터/Esc 처리)은 공용 UI 컴포넌트가, 단축키 바인딩은 전역 단축키 훅이 담당한다. 데스크탑 앱과 웹 대시보드가 동작이 동일하도록 같은 구조를 공유한다.

## 미확정 (OPEN)
- [ ] OPEN: 사이클(Cycle)은 화면 이동 대상으로 다뤄지나 현재 선택 종류에 사이클 케이스가 없어 팔레트에서의 사이클 이동은 미지원 — 향후 확장 여부 확인 필요
