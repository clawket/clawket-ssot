---
id: screen.help-modal
kind: Screen
title: 단축키 도움말 모달
definition: ? 키로 열리는 작은 모달로, 사용 가능한 키보드 단축키를 한 장의 치트시트로 보여준다.
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/HelpModal.tsx
  - desktop/apps/desktop/src/App.tsx
consumesApi: []
relatesTo:
  - to: screen.command-palette
    type: relates-to
    note: "대부분의 동작이 명령 팔레트에도 있다는 안내로 팔레트로 유도"
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적
사람 운영자가 키보드만으로 작업 공간을 빠르게 다루도록, 어떤 키가 무엇을 하는지 한 화면에서 확인하게 해 주는 참고용 모달이다.

## UI 요소 / 입력 필드
- 단축키 목록: 각 행에 동작 설명과 해당 키 조합(키 캡 형태)을 나란히 보여준다. 기본 항목은 ? (이 도움말 열기), ⌘+K (명령 팔레트 열기), Esc (모달·드로어 닫기)다.
- 닫기: 우상단 닫기 버튼, 배경 클릭, Esc 키로 닫는다.
- 안내 문구: "대부분의 동작은 명령 팔레트에서도 쓸 수 있다"는 팁으로 팔레트 사용을 권한다.

## 표시 데이터 / 호출 API
- 표시: 정적으로 정의된 단축키 목록(데이터 호출 없음).
- 호출: 외부 기능 호출이 없는 순수 표시용 모달이다.

## 상태 / 엣지케이스
- 닫힘 상태: 열림 플래그가 꺼지면 아무것도 렌더하지 않는다.
- 모달 내부 클릭은 닫힘으로 전파되지 않고, 배경(오버레이) 클릭만 닫는다.

## 미확정 (OPEN)
- [ ] OPEN: 단축키 목록이 정적 상수라 실제 등록된 전역 단축키(useGlobalShortcuts)와 항상 동기화되는지(불일치 가능성) 확인 필요.
