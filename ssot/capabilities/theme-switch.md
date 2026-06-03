---
id: capability.theme-switch
kind: Capability
title: 테마 전환(라이트/다크/시스템)
definition: 상단 바의 버튼으로 system→dark→light 를 순환 전환하고, system 일 때는 운영체제의 색 선호를 따르며, 선택은 브라우저에 저장돼 다음 방문에도 유지되는 기능이다.
servesPersona:
  - persona.builder
  - persona.human-operator
realizedBy:
  - component.web
  - component.web-app-shell
implementedIn:
  - web/src/lib/theme.ts
  - web/src/components/shell/Topbar.tsx
relatesTo:
  - to: component.web-app-shell
    type: belongs-to
    note: 앱 셸의 상단 바·전역 단축키 영역에 속하는 전역 UI 설정이다.
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
사용자는 대시보드의 화면 테마를 라이트·다크·시스템 세 가지로 바꾼다. 상단 바의 테마 버튼을 누를 때마다 system → dark → light 순서로 순환하며, 현재 어떤 선호가 걸렸고 실제로 어떤 색(밝게/어둡게)으로 보이는지를 버튼 라벨로 알 수 있다. "시스템"을 고르면 운영체제의 색 선호(prefers-color-scheme)를 그대로 따라가고, 사용자가 OS 설정을 바꾸면 화면도 즉시 같이 바뀐다. 선택은 브라우저에 저장돼 다음에 다시 와도 유지된다. 서버 호출이 전혀 없는 순수 클라이언트 설정이다.

## 행위
- 상단 바 버튼 클릭 시 테마 선호를 system→dark→light 로 순환한다.
- 선호를 브라우저 저장소(localStorage 키 `clawket.theme`)에 보존하고, 다음 방문 시 그 값으로 시작한다. 저장값이 없으면 기본은 system.
- 과거 키(`clawket-theme`, 대시 표기)에 light/dark 가 남아 있으면 첫 읽기에 새 키로 자동 이관한다.
- system 일 때는 OS 색 선호를 감지해 문서에 적용하고, OS 선호가 바뀌면 자동 반영한다.
- 다른 탭에서 테마가 바뀌면(저장소 이벤트) 현재 탭의 버튼 상태도 동기화한다.

## 시스템 흐름
앱이 처음 뜰 때 저장된 선호를 읽어 문서 루트의 테마 속성을 설정하고 OS 선호 변경 리스너를 건다. 사용자가 상단 바 버튼을 누르면 다음 테마로 순환해 저장하고 즉시 적용한다. system 선호일 때는 OS 변경 또는 다른 탭의 저장 변경에 반응해 실제 적용 색을 다시 계산한다. 모든 처리는 브라우저 안에서 끝나며 데몬과 통신하지 않는다.

## 어디에 구현되어 있나
테마 저장·적용·OS 선호 감지·레거시 키 이관 로직은 웹 앱의 테마 모듈(web/src/lib/theme.ts)에 있고, 순환 버튼과 라벨·아이콘은 상단 바 컴포넌트(web/src/components/shell/Topbar.tsx)에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 없음 (구현 경로 확인됨)
