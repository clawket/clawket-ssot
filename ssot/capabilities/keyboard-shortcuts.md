---
id: capability.keyboard-shortcuts
kind: Capability
title: 키보드 단축키
definition: Cmd/Ctrl+K로 명령 팔레트를, ?(물음표)로 단축키 도움말 모달을 여는 전역 키보드 단축키를 제공하며, 입력칸·텍스트영역 등에 글자를 치는 중에는 단축키가 발동하지 않도록 억제한다.
purpose: 사람이 마우스 없이 키보드만으로 앱의 핵심 진입점(팔레트·도움말)에 즉시 닿게 한다.
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/hooks/useGlobalShortcuts.ts
  - desktop/apps/desktop/src/shell/HelpModal.tsx
  - web/src/hooks/useGlobalShortcuts.ts
  - web/src/components/HelpModal.tsx
relatesTo:
  - { to: capability.command-palette, type: leads-to, note: Cmd/Ctrl+K가 여는 명령 팔레트 }
  - { to: screen.board, type: relates-to, note: 단축키가 전역으로 동작하는 대시보드 화면들 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 사용자가 할 수 있는 일

사람이 앱 어디에서나 Cmd/Ctrl+K를 누르면 명령 팔레트가, 물음표(?) 키를 누르면 단축키 도움말 모달이 뜬다. 도움말 모달은 사용할 수 있는 단축키 목록(도움말 열기, 팔레트 열기, Esc로 닫기)을 한눈에 보여주는 치트시트다. 단축키는 입력칸에 글자를 치는 중에는 발동하지 않아, 검색어나 본문에 "?"를 자연스럽게 입력할 수 있다.

## 행위

- Cmd/Ctrl+K → 명령 팔레트를 연다.
- ?(다른 보조키 없이) → 단축키 도움말 모달을 연다. 단, 입력칸·텍스트영역·선택상자·편집 가능한 영역에 포커스가 있으면 발동하지 않는다.
- Esc → 열려 있는 모달·드로어를 닫는다(도움말 모달에 안내되는 동작).
- 단축키는 화면 전역에 등록되어 어느 뷰에서든 동일하게 동작한다.

## 시스템 흐름

전역 키 입력 감지 → 눌린 조합이 팔레트(Cmd/Ctrl+K)인지 도움말(?)인지 판별 → 텍스트 입력 중이면 도움말 단축키는 억제 → 해당 핸들러로 팔레트 또는 도움말 모달을 연다.

## 어디에 구현되어 있나

전역 단축키 훅이 키 입력을 듣고 텍스트 입력 억제 규칙과 함께 팔레트·도움말 핸들러를 호출한다. 도움말 모달 컴포넌트가 단축키 치트시트를 그린다. 데스크탑 앱과 웹 대시보드가 동작이 같도록 동일한 훅 구조를 공유한다.

## 미확정 (OPEN)
- [ ] OPEN: 화면 전환용 숫자 단축키(예: ⌘1·⌘2 — 팔레트 뷰 항목 힌트에 표기됨)가 실제 키 바인딩으로 연결되어 있는지 확인 필요
