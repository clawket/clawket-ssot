---
id: capability.theme-toggle
kind: Capability
title: 테마 토글
definition: 상단바의 테마 버튼을 누를 때마다 밝게(light)·어둡게(dark)·시스템 설정 따름(system)을 차례로 순환시키고, 고른 값을 브라우저에 저장해 다음에 열 때도 유지하며 system일 때는 운영체제의 밝기 설정 변화를 자동으로 따라간다.
purpose: 사람이 자신의 눈과 환경에 맞게 화면 밝기를 고르고 그 선택이 계속 유지되게 한다.
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/Topbar.tsx
  - desktop/packages/ui/src/lib/theme.ts
  - web/src/components/shell/Topbar.tsx
relatesTo:
  - { to: screen.board, type: relates-to, note: 테마 토글이 놓인 상단바를 가진 대시보드 화면들 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 사용자가 할 수 있는 일

사람이 상단바의 테마 버튼을 누르면 화면 밝기 설정이 밝게 → 어둡게 → 시스템 설정 따름 순으로 한 단계씩 돌아간다. 버튼에는 현재 고른 값을 나타내는 아이콘(해·달·반달)이 표시된다. 고른 값은 브라우저에 저장되어 다음에 다시 열어도 그대로 유지되고, "시스템 설정 따름"을 골라 두면 운영체제가 밝은/어두운 모드로 바뀔 때 화면도 자동으로 함께 바뀐다.

## 행위

- 테마 버튼 클릭 → 밝게·어둡게·시스템 세 값을 순환한다.
- 고른 값은 브라우저 저장소에 보관되어 재방문 시 복원된다.
- "시스템 따름"일 때는 OS 다크모드 설정 변화를 감지해 화면을 자동으로 다시 칠한다.
- 여러 탭에서 같은 앱을 열어 둔 경우, 한 탭에서 바꾼 테마가 다른 탭에도 반영된다.

## 시스템 흐름

버튼 클릭 → 다음 테마 값 계산(순환) → 브라우저 저장소에 기록하고 문서의 테마 속성을 갱신해 즉시 다시 칠함. 시스템 값이면 OS 밝기 설정을 읽어 실제 적용할 밝기를 결정하고, OS 설정이 바뀌면 그 변화를 듣고 다시 적용한다.

## 어디에 구현되어 있나

상단바 컴포넌트가 테마 버튼과 순환 동작을 제공하고, 실제 저장·적용·OS 설정 추적 로직은 공용 테마 모듈이 담당한다. 데스크탑 앱과 웹 대시보드가 같은 테마 동작을 공유한다.

## 미확정 (OPEN)
- [ ] OPEN: 데스크탑(Tauri) 앱에서 OS 다크모드 연동이 웹과 동일하게 동작하는지(네이티브 창 테마 반영 포함) 확인 필요
