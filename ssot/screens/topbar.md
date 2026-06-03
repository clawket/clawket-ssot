---
id: screen.topbar
kind: Screen
title: 상단 바
definition: 데스크탑 워크벤치 상단의 가로 바로, 다섯 개 작업 뷰 사이를 탭으로 오가고 데몬 연결 상태를 알약 형태로 보여주며 테마 전환과 명령 팔레트 열기 버튼을 제공한다.
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/Topbar.tsx
consumesApi: [endpoint.daemon-health]
relatesTo:
  - to: screen.command-palette
    type: leads-to
    note: 명령 팔레트 열기 버튼이 팔레트를 띄운다
  - to: endpoint.daemon-health
    type: calls
    note: 데몬 헬스 알약이 끊김 시 즉시 재연결(재폴링)을 트리거
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적
사람 운영자가 작업 공간 안에서 보고 싶은 뷰(요약·보드·백로그·타임라인·위키)를 빠르게 전환하고, 백엔드 데몬이 살아 있는지 한눈에 확인하며, 화면 테마를 바꾸거나 명령 팔레트를 여는 상단 컨트롤 모음이다.

## UI 요소 / 입력 필드
- 뷰 탭: Summary(일일 스탠드업 랜딩), Board(상태별 칸반), Backlog(필터 가능한 작업 큐), Timeline(이벤트 스트림 리플레이), Wiki(프로젝트 마크다운 리더) 다섯 개를 탭으로 제공하며 현재 뷰가 강조된다.
- 데몬 헬스 알약: 데몬 연결 상태를 색과 점멸로 표시한다. 정상이면 녹색 "daemon ok", 끊기면 적색 점멸 "daemon down"으로 바뀌고 클릭하면 재연결(재폴링)을 시도한다.
- 테마 토글: 라이트/다크/시스템 세 가지 테마를 순환 전환하며 현재 선호 테마를 글리프로 표시한다.
- 명령 팔레트 열기 버튼: 누르면 검색형 명령 팔레트가 열린다(단축키 ⌘K 표기).

## 표시 데이터 / 호출 API
- 표시: 현재 활성 뷰, 데몬 연결 여부, 현재 테마 선호값.
- 호출: 데몬 헬스 상태는 데몬의 헬스 엔드포인트 폴링에 의존하며, 끊김 시 알약 클릭이 즉시 재폴링을 트리거한다. 뷰 전환·테마 전환·팔레트 열기는 셸 내부 상태 전환으로 처리되며 별도 데이터 호출이 없다.

## 상태 / 엣지케이스
- 데몬 정상: 알약은 비활성(클릭 불가)이며 안내 툴팁만 노출한다.
- 데몬 끊김: 알약이 클릭 가능 상태로 바뀌고, 클릭 시 헬스 재확인과 데이터 새로고침을 함께 수행한다.
- 활성 뷰 강조와 탭 역할(role=tab)로 접근성 상태가 노출된다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 끊김 시 재연결 폴링 주기·재시도 정책이 상단 바와 별도 헬스 훅 중 어디에 정의되는지 확인 필요.
