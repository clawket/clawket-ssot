---
id: domain.agent-runs
kind: Domain
title: 에이전트 실행 기록(Agent Runs)
purpose: "태스크별로 어떤 에이전트가 언제 실행되어 무엇을 바꿨는지를 기록·비교·재생해, 같은 작업의 여러 시도를 사후에 추적하고 모델 드리프트를 드러내는 영역이다."
definition: "태스크에 종속된 실행 기록(run)의 수명(시작/종료·에이전트·결과·변경 파일)과, 그 위에서 두 실행을 나란히 비교하거나 실행 이력을 시간순으로 재생하는 관찰 표면을 포괄한다. 태스크의 상태 전이 규칙 자체(task-management)나 실행을 허용/차단하는 훅 로직(hook-enforcement)은 이 영역 밖이다."
servesPersona: [persona.human-operator, persona.llm-coding-agent]
relatesTo:
  - to: domain.task-management
    type: relates-to
    note: "run 은 태스크에 종속된다. 태스크의 시작/완료가 run 을 만든다."
  - to: domain.hook-enforcement
    type: relates-to
    note: "실제 run 은 보통 Claude 훅이 태스크 시작 시 자동 생성하고, 변경 파일은 훅이 notes 에 적재한다."
  - to: domain.governance-audit
    type: relates-to
    note: "실행과 변경의 사후 추적이라는 점에서 감사 추적과 목적을 공유한다."
governedBy:
  - invariant.preserve-history
  - invariant.sse-payload-id-only
realizedBy: [component.web-dashboard, component.daemon, component.sse-hook]
impacts:
  - concept.run
  - concept.execution-envelope
  - concept.envelope-version-history
  - concept.timeline-event
  - capability.run-compare
  - capability.timeline-replay
  - capability.watch-replay
  - endpoint.daemon-events
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:domain"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 목적

같은 태스크는 여러 번 시도될 수 있다 — 다른 에이전트가, 다른 시점에, 다른 결과로. 이 영역은 그 시도 하나하나를 실행 기록으로 남겨 "누가 언제 돌렸고, 얼마나 걸렸고, 무엇을 바꿨고, 결과가 무엇이었는가" 를 사후에 되짚을 수 있게 한다. 두 실행을 나란히 놓아 차이를 보거나, 태스크의 실행 이력을 시간순 테이프로 재생해 상태가 어떻게 변해 왔는지 따라갈 수 있다. 핵심 가치는 사람 운영자가 에이전트 작업을 신뢰하기 위한 투명성이다.

## 경계와 핵심 개념

포함하는 개념:

- 실행 기록(concept.run): 태스크에 묶인 한 번의 실행. 시작/종료 시각, 실행한 에이전트(모델 프록시 식별자), 결과 문자열, 그리고 변경 파일 집합을 담는다. 변경 파일은 훅이 실행 중 포착해 기록에 적재한 것을 읽어 들인다.
- 실행 봉투(concept.execution-envelope)와 봉투 버전 이력(concept.envelope-version-history): 태스크에 서명된 실행 계약과 그 시간순 버전들. 실행 비교·재생 시 "그 실행이 시작된 순간 활성이던 봉투의 대상 모델" 을 해석하는 기준이 된다.
- 타임라인 이벤트(concept.timeline-event): 봉투 버전 변경과 실행의 시작/종료를 하나의 시간축 위에 올린 틱.

제외하는 것: 태스크의 상태 전이 규칙(domain.task-management), 실행을 허용/차단하는 훅 강제 로직(domain.hook-enforcement)은 이 영역의 책임이 아니다. 이 영역은 이미 일어난 실행을 "기록하고 보여 주는" 관찰 표면이다.

## 기능

- 실행 비교(capability.run-compare): 한 태스크의 모든 실행을 나열하고 정확히 둘을 골라 나란히 차이를 본다. 실행이 직접 들고 있는 정보(에이전트·소요 시간·결과·변경 파일)와, 각 실행 시작 시점에 활성이던 봉투의 대상 모델을 함께 보여 줘 모델 드리프트를 드러낸다.
- 타임라인 재생(capability.timeline-replay): 봉투 버전 변경과 실행 시작/종료를 시간순으로 합쳐 하나의 테이프로 만들고, 슬라이더로 어느 순간의 상태든 되짚거나 자동 재생한다.
- 실시간 관찰·리플레이(capability.watch-replay): 데몬이 푸시하는 실행 생성/갱신 이벤트를 구독해, 폴링 없이 실행 관련 화면을 즉시 갱신한다.

## 시스템 흐름

태스크가 시작되면(보통 훅이 자동으로) 실행 기록이 열리고 시작 시각·에이전트가 새겨진다. 작업이 끝나면 종료 시각·결과가 채워지고, 실행 중 바뀐 파일 목록이 기록의 노트에 담긴다. 관찰 측에서는 웹 대시보드가 데몬의 실시간 이벤트 스트림을 구독해, 해당 태스크의 실행이 생기거나 갱신될 때마다 실행 목록과 봉투 이력을 다시 불러와 비교·재생 화면을 살아 있게 유지한다. 비교 화면은 두 실행을 골라 변경 파일을 "한쪽에만 있는 것 / 양쪽 공통" 으로 나눠 보여 주고, 재생 화면은 봉투 버전과 실행 이벤트를 하나의 시간축에 정렬해 슬라이더로 훑게 한다.

## 다른 도메인과의 관계

- domain.task-management 와는 종속 관계다 — 실행 기록은 태스크에 매달려 있고, 태스크의 시작/완료가 실행 기록을 만든다.
- domain.hook-enforcement 와는 생성 출처에서 만난다 — 실제 실행 기록과 변경 파일 데이터는 대개 Claude 훅이 태스크 시작 시 자동으로 만들고 적재한다.
- domain.governance-audit 와는 "실행과 변경의 사후 추적" 이라는 목적을 공유한다.

## 미확정 (OPEN)
- [ ] OPEN: 변경 파일이 별도 구조화 컬럼 없이 실행 노트의 JSON 블롭을 방어적으로 파싱해 얻어지는 현재 구조가 의도된 영구 설계인지, 추후 정규화 대상인지 확정 필요(RunCompare 의 parseChangedFiles 주석 근거).
- [ ] OPEN: 실시간 이벤트가 id 만 싣고 페이로드 본문은 비우는 계약(invariant.sse-payload-id-only)이 본 영역의 모든 실시간 갱신에 일관 적용되는지 확정 필요.
