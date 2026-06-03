---
id: capability.timeline-replay
kind: Capability
title: 타임라인 리플레이 재생
definition: 한 태스크의 작업 계약(봉투) 버전 변화와 에이전트 실행(run) 시작·종료를 하나의 시간순 테이프로 합쳐, 슬라이더로 특정 순간으로 되감거나 자동 재생하며 그 시점의 상태를 되짚어 보는 기능이다.
servesPersona:
  - persona.builder
  - persona.human-operator
realizedBy:
  - component.web
  - component.web-api-client
implementedIn:
  - web/src/features/timeline/TimelineReplay.tsx
  - web/src/api.ts
relatesTo:
  - to: concept.run
    type: reads
    note: 각 run 의 시작·종료 시각과 에이전트·결과를 테이프의 이벤트로 표시한다.
  - to: concept.envelope-version-history
    type: reads
    note: 봉투 버전 이력을 시간축의 또 다른 이벤트 줄기로 합친다.
  - to: concept.envelope
    type: relates-to
    note: 슬라이더가 멈춘 순간에 활성이던 봉투 상태(JSON)를 카드로 보여준다.
  - to: screen.web-timeline
    type: belongs-to
    note: 웹 Timeline 화면 안에서 태스크별 리플레이 표면으로 제공된다.
  - to: domain.agent-runs
    type: belongs-to
    note: 에이전트 실행 추적 영역의 사후 점검 표면이다.
impacts:
  - domain.agent-runs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. -->

## 사용자가 할 수 있는 일
사용자(1인 빌더 또는 사람 운영자)는 한 태스크가 시간에 따라 어떻게 변해 왔는지를 영상처럼 되감아 볼 수 있다. 태스크의 작업 계약(봉투)이 몇 번 바뀌었는지, 각 시점에 어떤 에이전트가 실행을 시작하고 끝냈는지를 하나의 흐름으로 합쳐, 슬라이더를 좌우로 움직이거나 자동 재생으로 따라가며 "그 순간 무엇이 활성 상태였는지"를 확인한다. 차트 라이브러리 없이 슬라이더 + 그 시점 상태 카드만으로 검증에 필요한 만큼의 명료함을 제공하려는 의도다.

## 행위
- 태스크의 봉투 버전 이력과 실행(run) 목록을 동시에 읽어온다.
- 두 줄기를 시각 기준 오름차순으로 합쳐 하나의 이벤트 테이프(틱 0..N-1)로 만든다. 같은 순간이면 봉투 변화가 실행 이벤트보다 앞에 온다.
- 슬라이더로 임의의 틱으로 이동하거나, 재생/일시정지로 1초에 한 틱씩 자동 전진한다(마지막 틱에서 멈춤).
- 현재 틱의 이벤트(봉투 변경 / 실행 시작 / 실행 종료)와, 그 순간 활성이던 봉투의 전체 상태를 보여준다.
- 처음 열면 가장 최신 틱(현재 상태)에 위치하고, 거기서 과거로 되감는 식으로 쓴다.

## 시스템 흐름
태스크 상세에서 리플레이 표면이 열리면, 봉투 버전 이력과 실행 목록을 데몬 HTTP API로 한꺼번에 가져온다(웹 API 클라이언트 경유). 두 데이터를 시간순으로 병합해 테이프를 구성하고, 슬라이더 위치(틱)가 바뀔 때마다 현재 이벤트와 "그 시점 이전의 가장 최근 봉투"를 거꾸로 거슬러 찾아 활성 봉투로 해석해 카드에 렌더링한다. 봉투 버전도 실행도 없으면 "되짚을 이력이 아직 없다"고 안내한다.

## 어디에 구현되어 있나
웹 앱의 타임라인 기능 컴포넌트(web/src/features/timeline/TimelineReplay.tsx)가 슬라이더·재생·상태 카드를 그리고, 봉투 이력·실행 목록 조회는 데몬 API 래퍼(web/src/api.ts)를 통한다. 웹 Timeline 화면의 일부로 노출된다.

## 미확정 (OPEN)
- [ ] OPEN: 봉투 이력/실행 목록을 실제로 서빙하는 데몬 엔드포인트 노드와의 calls/reads 엣지 정렬(인벤토리에 web 스코프 전용 봉투-이력 엔드포인트 노드 미확인)
