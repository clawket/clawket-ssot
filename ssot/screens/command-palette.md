---
id: screen.command-palette
kind: Screen
title: 명령 팔레트
definition: ⌘/Ctrl+K로 열리는 검색형 오버레이로, 한 입력창에서 작업 뷰·플랜·유닛·작업을 그룹별로 검색해 해당 위치로 점프한다.
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/App.tsx
  - desktop/packages/ui/src/components/CommandSurface/CommandSurface.tsx
consumesApi: [endpoint.daemon-plans, endpoint.daemon-units, endpoint.daemon-tasks]
relatesTo:
  - to: concept.selection
    type: leads-to
    note: 플랜·유닛·작업을 고르면 선택을 바꿔 상세 드로어를 띄운다
  - to: concept.ticket-number
    type: relates-to
    note: 작업 항목에 사람이 읽는 티켓 번호를 함께 표시
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적
사람 운영자가 마우스로 트리를 뒤지지 않고도, 키보드 한 번으로 작업 공간 어디로든 빠르게 이동하는 점프 도구다. 뷰 전환, 특정 플랜·유닛·작업으로의 이동을 한 검색창에서 통합 처리한다.

## UI 요소 / 입력 필드
- 검색 입력창: 열리면 자동으로 포커스되며, 입력값으로 결과를 즉시 거른다.
- 그룹별 결과 목록: Views / Plans / Units / Tasks 그룹으로 묶여 표시된다. 뷰 항목에는 ⌘1~⌘5 같은 단축 힌트가, 플랜에는 상태가, 작업에는 티켓 번호와 상태가 함께 보인다.
- 키보드 조작: 위/아래 화살표로 항목 이동(순환), Enter로 활성 항목 실행, Esc 또는 배경 클릭으로 닫는다.
- 마우스 조작: 항목 위로 마우스를 올리면 활성 표시가 따라오고 클릭하면 실행된다.

## 표시 데이터 / 호출 API
- 표시: 다섯 개 뷰, 그리고 현재 프로젝트의 플랜·유닛·작업 목록.
- 검색·정렬: 입력이 없을 때는 목록을 그룹별로 일정 개수까지만 잘라 스캔하기 쉽게 보여주고, 입력이 있으면 항목 라벨과 작업 티켓 번호에 대해 부분 일치로 거른다.
- 실행 동작: 뷰 항목은 활성 뷰를 전환하고, 플랜·유닛·작업 항목은 해당 항목을 선택해 상세 드로어로 이끈다. 선택 후 팔레트는 닫히고 검색어가 초기화된다.

## 상태 / 엣지케이스
- 빈 결과: 일치 항목이 없으면 "일치 항목 없음"을 표시한다.
- 닫힘 상태: 열림 플래그가 꺼지면 아무것도 렌더하지 않는다.
- 사이클 점프: 현재 선택 모델이 사이클 종류를 아직 포함하지 않아, 사이클로의 직접 점프는 제공되지 않는다(작업 환경상 뷰 점프 용도로만 노출되는 한계).

## 미확정 (OPEN)
- [ ] OPEN: 데스크탑 앱과 웹(별도 CommandPalette 구현)의 팔레트 동작·그룹 구성이 동일한지(파리티) 확인 필요.
