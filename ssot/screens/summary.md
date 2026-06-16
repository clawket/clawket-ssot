---
id: screen.summary
kind: Screen
title: Summary 화면 (데일리 스탠드업)
purpose: "사람 운영자가 프로젝트의 현재 상태를 한 눈에 파악한다 — 전체 진행률, 상태별 작업 수, 지금 진행 중인 작업, 최근 활동을 데일리 스탠드업처럼 본다."
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/views/SummaryView.tsx
  - desktop/apps/desktop/src/views/ViewShell.tsx
  - web/src/components/SummaryView.tsx
  - web/src/components/SummaryView.helpers.ts
consumesApi: [endpoint.daemon-events]
relatesTo:
  - { to: concept.task, type: reads, note: "상태별 작업 수와 진행 중 작업 카드를 태스크에서 집계한다" }
  - { to: concept.plan, type: reads, note: "active 플랜의 제목을 부제로 표시한다" }
  - { to: concept.cycle, type: reads, note: "active 사이클이 속한 유닛 제목을 'Now active' 헤더에 표시한다" }
  - { to: concept.tier, type: reads, note: "진행 중 작업 카드에 티어 마크를 표시한다" }
  - { to: capability.web-review, type: realizes, note: "5개 뷰 중 상태 검토 진입 화면" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Summary 화면은 사람 운영자가 프로젝트의 작업 상태를 데일리 스탠드업처럼 훑어보는 랜딩 화면이다. 부제(subtitle)에는 현재 active 상태인 플랜의 제목이 나오고, active 플랜이 없으면 "No active plan"이라고 표시한다. 화면 하나에서 "이 프로젝트가 얼마나 진행됐고, 지금 무엇이 돌아가고 있으며, 최근에 무슨 일이 있었는지"를 답한다.

여기서 "active"의 의미는 최신성이 아니라 라이프사이클 상태다. 가장 최근에 만들어진 플랜이 아니라 실제로 active 상태인 플랜만 활성으로 표시하며, 그런 플랜이 없으면 빈 상태를 보여준다. 진행 중 작업도 마찬가지로 실제 in_progress 상태인 태스크만 활성으로 본다 — 이는 화면이 프로젝트의 진짜 상태를 거짓 없이 보여주기 위한 규칙이다.

## UI 요소 / 입력 필드

이 화면은 읽기 전용 대시보드로, 입력 필드는 없다. 다음 정보 블록으로 구성된다.

- 전체 진행률 카드: 완료/진행 중/대기/차단 작업 비율을 막대로 보여주고, 완료 비율을 퍼센트로 표시한다. 차단된 작업이 있을 때만 차단 범례가 추가로 나타난다.
- 상태별 KPI 스트립: todo · in_progress · blocked · done · cancelled 다섯 상태 각각의 작업 수를 카드로 나열한다.
- "Now active" 영역: 현재 진행 중인 대표 작업을 카드로 보여준다(티켓 번호, 제목, 본문 요약, 상태, 담당 에이전트, 근거 보유 여부, 마지막 갱신 시각). 같은 진행 중 작업이 더 있으면 그 아래 간단한 행 목록으로 추가 표시한다.
- "Recent activity" 타임라인: 최근 활동 이벤트 5건을 시간 역순으로 보여주고, 각 이벤트에 변경 유형(생성/시작/완료/취소/차단 등)에 따라 색 점을 찍는다.

## 표시 데이터 / 호출 API

화면이 보여주는 데이터는 프로젝트의 플랜·유닛·사이클·태스크 목록과 활동 타임라인이다. 작업 수 집계, active 플랜·진행 중 작업 판별, active 사이클이 속한 유닛 제목 찾기는 모두 이 데이터를 클라이언트에서 가공해 도출한다.

데이터는 데몬이 공급하며, 데스크탑·웹 모두 데몬의 실시간 이벤트 스트림(SSE)을 구독하는 공통 데이터 계층(DataProvider 등)을 통해 받는다. 즉 다른 화면에서 작업 상태가 바뀌면 이 화면의 집계도 함께 갱신된다.

## 상태 / 엣지케이스

- 로딩/대기 상태: "Loading data…"를 보여준다.
- 에러 상태: 데이터 로드 실패 메시지를 표시한다.
- active 플랜 없음: 부제가 "No active plan"이 된다.
- 진행 중 작업 없음: "No tasks in this project yet." 안내를 보여준다.
- 최근 활동 없음: "No activity yet." 안내를 보여준다.

## 미확정 (OPEN)
- [ ] OPEN: 데스크탑 SummaryView와 web SummaryView 사이의 표시 항목 차이(있다면)를 정밀 대조해 자연어로 명시할 것
