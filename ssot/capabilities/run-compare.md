---
id: capability.run-compare
kind: Capability
title: 두 실행(run) 비교
definition: 한 태스크의 여러 실행(run) 중 정확히 두 개를 골라, 어떤 에이전트가 얼마나 걸려 어떤 결과를 냈고 어떤 파일을 바꿨는지를 좌우로 나란히 놓고 차이를 보는 기능이다.
servesPersona:
  - persona.builder
  - persona.human-operator
realizedBy:
  - component.web
  - component.web-api-client
implementedIn:
  - web/src/features/runs/RunCompare.tsx
  - web/src/api.ts
relatesTo:
  - to: concept.run
    type: reads
    note: 비교 대상은 태스크의 실행(run) 기록 — 에이전트·소요시간·결과·변경 파일이다.
  - to: concept.envelope-version-history
    type: reads
    note: 각 실행이 시작된 순간 활성이던 봉투의 target_model 을 봉투 이력에서 거슬러 찾아 모델 드리프트를 함께 보여준다.
  - to: domain.agent-runs
    type: belongs-to
    note: 에이전트 실행 추적·비교 영역의 핵심 표면이다.
  - to: screen.web-timeline
    type: relates-to
    note: 에이전트별 실행을 다루는 Timeline 맥락과 짝을 이루는 실행 비교 표면.
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

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 사용자가 할 수 있는 일
사용자는 같은 태스크가 여러 번 실행됐을 때 두 번의 실행을 골라 무엇이 달랐는지 한눈에 비교할 수 있다. 어느 에이전트가 돌렸는지, 얼마나 걸렸는지, 결과가 무엇이었는지, 그리고 각 실행이 건드린 파일이 무엇이고 둘 다 건드린 공통 파일은 무엇인지를 좌우 패널로 본다. 같은 태스크라도 봉투가 중간에 바뀌면 실행마다 대상 모델(target_model)이 달라질 수 있으므로, 실행 시작 시점의 봉투 모델까지 함께 보여 모델 드리프트를 드러낸다.

## 행위
- 태스크의 모든 실행을 목록으로 보여주고, 체크박스로 정확히 두 개를 선택한다(세 번째를 고르면 가장 오래된 선택이 밀려난다).
- 선택된 두 실행에 대해 에이전트, 소요시간(종료−시작, 진행 중이면 "running"), 결과, 변경 파일 집합을 좌우로 표시한다.
- 변경 파일은 각 실행의 노트에 담긴 JSON(`changed_files` 배열)에서 최선의 노력으로 파싱한다. 파싱 불가하면 "보고된 변경 파일 없음"으로 처리한다.
- 한쪽에만 있는 파일과 양쪽 공통 파일을 구분해 보여준다.
- 실행 시작 시각 기준으로 봉투 이력을 거슬러 그 순간의 target_model 을 각 실행 패널에 함께 표시한다.
- 데몬이 이 태스크에 대한 실행 생성/갱신을 알리면 폴링 없이 목록과 비교가 실시간으로 갱신된다.

## 시스템 흐름
태스크 상세에서 비교 표면이 열리면 실행 목록과 봉투 이력을 데몬 API로 함께 가져온다. 사용자가 두 실행을 고르면 각 실행의 노트를 파싱해 변경 파일을 추출하고, 시작 시각으로 봉투 이력을 거슬러 활성 모델을 해석해 좌우 패널과 공통 파일 패널을 그린다. 실행이 2개 미만이면 비교 불가 안내를 띄운다. 실행 이벤트 구독으로 목록·비교가 살아 있는 상태로 유지된다.

## 어디에 구현되어 있나
웹 앱의 실행 비교 컴포넌트(web/src/features/runs/RunCompare.tsx)가 선택 목록·좌우 diff·공통 파일 패널을 그리고, 실행 목록·봉투 이력 조회는 데몬 API 래퍼(web/src/api.ts)를 통한다. 실시간 갱신은 실행 이벤트 훅을 구독한다.

## 미확정 (OPEN)
- [ ] OPEN: 실행 목록/봉투 이력을 서빙하는 데몬 엔드포인트 노드와의 reads 엣지 정렬(web 스코프 전용 노드 미확인 — daemon.runs 는 desktop 스코프)
