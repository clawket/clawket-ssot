---
id: screen.web-timeline
kind: Screen
title: Timeline 화면 (스윔레인·활동 스트림)
definition: 에이전트별 실행 막대를 시간축 스윔레인으로 시각화하고, 별도 탭으로 프로젝트 활동 이벤트를 시간순 스트림으로 보여주는 화면. 동시에 돌아간 에이전트 작업을 한눈에 본다.
purpose: 사람 운영자가 여러 에이전트의 실행(run)을 시간축 스윔레인으로 비교하고 활동 이벤트 흐름을 추적한다
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard]
implementedIn:
  - web/src/components/TimelineView.tsx
consumesApi:
  - endpoint.daemon-runs
  - endpoint.daemon-timeline
  - endpoint.daemon-cycles
  - endpoint.daemon-tasks
  - endpoint.daemon-events
relatesTo:
  - { to: capability.web-review, type: realizes, note: 5개 웹 뷰 중 실행 시각화 표면 }
  - { to: capability.watch-replay, type: reads, note: 실행 이력의 사후 관찰과 같은 데이터를 시각화 }
  - { to: concept.run, type: reads, note: 에이전트별 run을 시간 막대로 표현 }
  - { to: concept.task, type: reads, note: run을 태스크 제목·티켓에 연결하고 차단/의존 관계 표시 }
  - { to: concept.cycle, type: reads, note: 활성 사이클 진행 게이지 표시 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Timeline 화면은 여러 에이전트가 동시에 또는 시차를 두고 수행한 실행(run)을 시간축 위에 시각화해, 사람 운영자가 "누가 언제 무엇을 얼마나 오래 돌렸는지"를 비교 관찰하도록 한다. 스윔레인 탭은 에이전트별 수직 트랙에 실행 막대를 그려 동시 작업과 소요 시간을 직관적으로 드러내고, 활동(Activity) 탭은 프로젝트 전체 이벤트를 날짜별 시간순 스트림으로 보여준다. 활성 사이클이 있으면 그 진행률 게이지를 상단에 함께 표시한다.

## UI 요소 / 입력 필드

- **탭 전환기**: Swimlane / Activity 두 탭을 전환한다.
- **사이클 진행 게이지**: 활성 사이클의 완료/진행중/차단 비율을 색 막대와 수치(완료 개수, %, 시작일, 대략적 잔여 추정)로 표시한다.
- **스윔레인(Swimlane) 탭**: 에이전트별 세로 트랙을 나란히 배치하고, 각 트랙에 실행 막대를 시작·종료 시각 비율로 배치한다. 막대 색은 실행 결과(성공/실패/세션종료/실행중)를 의미하며, 호버하면 강조되고 막대 클릭 시 해당 태스크 상세로 이동한다. 왼쪽에 시간 눈금 축, 상단에 에이전트 이름과 실행 수, 하단에 차단/의존 태스크 목록·범례·호버 상세 패널을 둔다.
- **활동(Activity) 탭**: 이벤트를 날짜(오늘/어제/날짜) 그룹으로 묶어 각 이벤트의 아이콘·행위 설명·대상·세부(상태 변화, 코멘트 일부, 실행 결과·소요시간 등)·시각을 행으로 보여준다. 태스크 이벤트 클릭 시 해당 태스크 상세로 이동한다.

직접 입력 필드는 없고 탭 선택·호버·클릭 상호작용만 제공한다.

## 표시 데이터 / 호출 API

진입 시 프로젝트의 실행 목록·프로젝트 타임라인 이벤트·사이클 목록을 함께 불러오고, 활성 사이클이 있으면 그 사이클의 태스크로 진행 게이지를 구성한다. 실행 막대에 제목을 붙이기 위해 실행에 연결된 태스크들을 추가로 조회한다(상한 내). 데몬의 실행·타임라인·사이클·태스크 조회 API를 사용한다. App 상위의 SSE 태스크 델타 계약을 받지만 현재 재렌더는 상위 마운트 키로 구동된다.

종료 시각이 없는 실행은 최근 시작이면 "실행 중"으로, 오래 방치된 것은 "세션 종료" 스냅샷으로 처리해 거대한 막대가 전체 레인을 가리지 않도록 보정한다.

## 상태 / 엣지케이스

- **로딩 중**: "Loading timeline..." 표시.
- **실행 없음**: 스윔레인 탭에 "No runs yet. Agent executions will appear here." 안내.
- **활동 없음**: 활동 탭에 "No activity yet." 안내.
- **고아 실행(종료 누락)**: 일정 시간(1시간) 넘게 닫히지 않은 실행은 세션 종료로 간주해 시각화 왜곡을 방지.
- 차단/의존 태스크가 있을 때만 하단 의존 패널을 노출하며, 차단 태스크는 무엇에 막혔는지 함께 표시한다.

## 미확정 (OPEN)
- [ ] OPEN: 실행 결과(result) 값 어휘 전체와 색 매핑을 자연어로 확정
- [ ] OPEN: 잔여 시간 추정 계산식의 의미와 신뢰도 한계를 명시
