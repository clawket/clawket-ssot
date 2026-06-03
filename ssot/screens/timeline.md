---
id: screen.timeline
kind: Screen
title: Timeline 화면 (스윔레인 / 활동 스트림)
purpose: "사람 운영자가 에이전트별 실행 트랙(스윔레인)과 활동 이벤트 스트림을 보고, 활성 사이클의 진행 밴드와 차단·의존 관계를 점검하며 사후에 무슨 일이 있었는지 재생한다."
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/views/TimelineView.tsx
  - desktop/apps/desktop/src/views/timeline/Swimlane.tsx
  - desktop/apps/desktop/src/views/timeline/ActivityStream.tsx
  - desktop/apps/desktop/src/views/timeline/CycleProgress.tsx
  - web/src/components/TimelineView.tsx
consumesApi: [endpoint.daemon-events]
relatesTo:
  - { to: concept.run, type: reads, note: "스윔레인은 태스크 실행 기록(run)을 에이전트별 트랙으로 펼친다" }
  - { to: concept.cycle, type: reads, note: "활성 사이클의 진행 밴드를 상단에 보여준다" }
  - { to: concept.task, type: reads, note: "차단된 작업과 depends_on 관계를 의존 패널에 표시한다" }
  - { to: concept.ticket-number, type: reads, note: "run·차단 작업에 티켓 번호를 함께 보여준다" }
  - { to: capability.web-review, type: realizes, note: "실행 이력 검토 뷰" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Timeline 화면은 프로젝트에서 "누가(어떤 에이전트가) 언제 무엇을 실행했는지"를 두 가지 관점으로 보여주는 이벤트 재생 화면이다. 스윔레인 탭은 에이전트별 실행 트랙을, 활동(Activity) 탭은 시간순 이벤트 스트림을 보여준다. 상단에는 활성 사이클이 있을 때 그 진행 상황을 막대 밴드로 표시한다. 부제에는 현재 탭에 맞춰 run 수 또는 이벤트 수가 나온다.

## UI 요소 / 입력 필드

- 탭 스위처: "Swimlane"과 "Activity" 두 탭을 전환한다.
- 활성 사이클 진행 밴드: 활성 사이클이 있고 그 작업 목록을 받아오면, 상태별 진행을 세그먼트 막대와 캡션으로 보여준다.
- 스윔레인 탭: 에이전트별 세로 실행 트랙. run에 마우스를 올리면 작업 제목·티켓 등 상세를 보여준다. 트랙 아래에 의존/차단 패널이 있어, 활성 사이클에 차단된 작업이나 depends_on 간선을 가진 작업이 있으면 그 목록과 "무엇에 막혀 있는지"를 보여준다.
- 활동 탭: 날짜별로 묶인 이벤트 스트림을 아이콘·시각과 함께 보여준다.
- 작업 클릭: run·이벤트·차단 행에서 작업을 클릭하면 상세를 연다(데스크탑은 DetailDrawer, 웹은 라우트 이동).

## 표시 데이터 / 호출 API

화면은 실행 기록(runs)·활동 타임라인·태스크·사이클 목록을 받는다. 데스크탑은 직접 API를 호출하지 않고 공통 데이터 계층(DataProvider)을 통해 받으며, 이 계층이 run과 타임라인을 데몬의 실시간 이벤트 스트림(SSE)으로 흘려보낸다. 스윔레인의 run에는 캐시된 태스크 목록에서 제목·티켓을 붙여 추가 조회 없이 표시한다.

활성 사이클의 진행 밴드는 그 사이클에 속한 작업 목록을 필요할 때 별도로 받아 계산한다(사이클 작업은 이 화면에서만 쓰므로 캐시하지 않고 사이클 id로 그때그때 조회). 사이클 작업 상태의 진실은 데몬이 가지므로, 전체 태스크 목록에서 임의로 유추하지 않는다.

## 상태 / 엣지케이스

- 로딩/에러 상태: 각각 "Loading data…", 에러 메시지를 표시한다.
- 활성 사이클 없음 또는 사이클 작업 조회 실패: 진행 밴드를 렌더하지 않고 나머지 화면은 유지한다(소프트 페일).
- 차단·의존 없음: 의존/차단 패널을 표시하지 않는다.
- 차단 작업만 있을 때는 차단 목록과 차단 원인 작업을, 의존만 있을 때는 의존 작업 수 요약을 보여준다.

## 미확정 (OPEN)
- [ ] OPEN: 스윔레인의 시간축 표현과 ActivityStream의 이벤트 분류 규칙을 코드로 더 정독해 자연어로 보강할 것
