---
id: screen.web-summary
kind: Screen
title: Summary 화면 (프로젝트 개요)
definition: 활성 프로젝트의 전체 진행도와 지금 진행 중인 작업, 최근 활동을 한눈에 보여주는 웹 대시보드의 개요 화면. 사람 운영자가 프로젝트 상태를 빠르게 파악하는 진입점이다.
purpose: 사람 운영자가 프로젝트의 전체 진행률·상태별 작업 분포·지금 활성인 작업·최근 활동을 한 화면에서 파악한다
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard]
implementedIn:
  - web/src/components/SummaryView.tsx
  - web/src/components/SummaryView.helpers.ts
consumesApi:
  - endpoint.daemon-plans
  - endpoint.daemon-units
  - endpoint.daemon-tasks
  - endpoint.daemon-cycles
  - endpoint.daemon-timeline
relatesTo:
  - { to: capability.web-review, type: realizes, note: 5개 웹 뷰 중 상태 검토 표면 }
  - { to: concept.task, type: reads, note: 상태·티어·근거·담당 에이전트를 카드로 표시 }
  - { to: concept.cycle, type: reads, note: 활성 사이클이 속한 유닛 제목을 헤더에 표시 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Summary 화면은 사람 운영자가 웹 대시보드를 열었을 때 프로젝트의 현재 상태를 가장 빠르게 파악하도록 돕는 개요 화면이다. 한 프로젝트 안의 모든 플랜·유닛·태스크·사이클을 종합해 "전체 어디까지 왔는지", "지금 무엇이 돌고 있는지", "방금 무슨 일이 있었는지"를 한 페이지로 요약한다. 개별 작업의 세부 편집은 다른 화면(Board·Wiki 등)으로 넘기고, 이 화면은 의사결정에 필요한 상태 신호만 압축해 보여주는 역할에 집중한다.

## UI 요소 / 입력 필드

화면 상단에는 현재 활성 플랜의 제목이 부제로 표시되며, 활성 플랜이 없으면 그 사실을 명시한다. 그 아래에 다음 요소들이 배치된다.

- **전체 진행도 카드**: 닫힌 작업(완료+취소) 비율을 백분율로 보여주고, 닫힘·진행중·할일·차단 구간을 색으로 나눈 진행 막대와 각 구간의 개수를 함께 표시한다.
- **상태별 KPI 묶음**: todo·in_progress·blocked·done·cancelled 다섯 상태별 작업 개수를 카드로 나열한다.
- **"지금 활성" 영역**: 현재 활성인 대표 작업 하나를 큰 카드로 강조해 제목·본문 요약·상태·담당 에이전트·근거(evidence) 유무·티어·최종 갱신 시각을 보여주고, 그 외 진행 중인 작업들을 간단한 행 목록으로 덧붙인다. 활성 사이클이 속한 유닛 제목도 함께 노출한다.
- **최근 활동 영역**: 프로젝트 타임라인에서 가져온 최근 이벤트(상태 변경·코멘트·실행 시작/종료·지식 추가 등) 다섯 건을 시간순 목록으로 보여준다.

사용자의 직접 입력 필드는 없고, 작업 카드나 활동 행을 클릭하면 해당 태스크 상세로 이동하는 선택 동작만 제공한다.

## 표시 데이터 / 호출 API

이 화면은 진입 시 대상 프로젝트의 플랜 목록·사이클 목록·프로젝트 타임라인을 먼저 불러온 뒤, 각 플랜의 유닛을 조회하고 다시 각 유닛의 태스크를 모아 전체 작업 집합을 구성한다. 즉 플랜→유닛→태스크 계층을 펼쳐 프로젝트 전역 작업 통계를 만든다. 데몬의 플랜·유닛·태스크·사이클·타임라인 조회 API를 사용한다.

진행률은 닫힌 작업 수를 전체로 나눠 산출하며, 활성 플랜·활성 태스크 판정은 보조 헬퍼 로직이 담당한다. 근거 유무 칩은 done 전환에 근거가 필요하다는 불변식을 사용자에게 시각적으로 환기한다(근거 미첨부 시 경고 색).

## 상태 / 엣지케이스

- **로딩 중**: 데이터 적재 동안 "Loading summary..." 표시.
- **활성 플랜 없음**: 부제에 "No active plan"을 노출.
- **작업 없음**: "지금 활성" 영역에 아직 작업이 없다는 안내를 표시.
- **활동 없음**: 최근 활동 영역에 "No activity yet" 안내.
- 타임라인 조회가 실패하면 빈 목록으로 우아하게 폴백해 화면 자체는 계속 렌더링된다.

## 미확정 (OPEN)
- [ ] OPEN: 활성 태스크 판정 헬퍼(SummaryView.helpers)의 우선순위 규칙을 자연어로 명시
- [ ] OPEN: SSE 실시간 갱신이 이 화면에 어떻게 반영되는지(자동 새로고침 여부) 확인
