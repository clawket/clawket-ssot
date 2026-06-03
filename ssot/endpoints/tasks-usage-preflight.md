---
id: endpoint.tasks-usage-preflight
kind: Endpoint
title: 태스크 사용량 사전점검
definition: "GET /tasks/{id}/usage/preflight — 태스크의 누적 토큰·비용이 예산 안이면 통과(200), 초과면 차단(429)으로 응답한다."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/usage.rs]
relatesTo:
  - { to: concept.usage, type: reads, note: "태스크에 기록된 누적 사용량 합계를 읽는다" }
  - { to: concept.task-envelope, type: reads, note: "실행 봉투 상속 체인에서 토큰 예산을 해석한다" }
  - { to: concept.task, type: reads, note: "대상 태스크 존재 확인" }
governedBy: []
impacts: [concept.task]
consumedBy: [endpoint.hook-pre-tool-use]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크에는 토큰·비용 예산이 걸릴 수 있다. 이 동작은 "지금 이 태스크에서 작업을 더 진행해도 되는가"를 작업 직전에 빠르게 묻는 신호등이다. 누적 사용량이 예산 안에 있으면 통과를, 한도를 넘었으면 "더는 진행하지 말라"는 차단 신호를 돌려준다. 도구 사용 직전 가드(PreToolUse 훅)가 이 신호를 보고, 예산을 다 태운 태스크에서 에이전트가 계속 토큰을 쓰는 것을 막는 용도로 쓰도록 설계됐다.

## 요청 / 응답
요청은 대상 태스크를 가리키기만 하면 된다. 응답으로는 예산 대비 현황 보고서를 돌려주는데, 여기에는 적용된 예산(없으면 비어 있음), 지금까지 쓴 양, 그리고 입력 토큰·출력 토큰·비용 각각이 한도를 넘었는지 여부와 "하나라도 넘었는가"라는 종합 판정이 담긴다.

예산 안이면 정상 통과로 응답하고, 한 항목이라도 초과하면 차단 응답("예산 초과")으로 같은 보고서를 함께 돌려준다. 가드는 이 차단 응답을 예산을 올리기 전까지의 강제 정지로 해석한다. 예산이 아예 설정돼 있지 않으면 초과가 없는 것으로 보아 통과한다. 대상 태스크가 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약
예산은 태스크 자체가 아니라 실행 봉투의 토큰 예산 항목에서 읽으며, 상속 체인을 따라 깊은 단계(자식)의 값이 부모 값을 덮어쓴다. 즉 상위 태스크에 건 예산이 별도 지정이 없는 한 모든 하위 태스크로 흘러내린다. 사용량 기록 자체는 기록 당시의 예산을 스냅샷으로 함께 남기므로, 나중에 예산을 고쳐도 과거 기록의 판정 기준이 바뀌지는 않는다.

## provenance
구현은 데몬의 토큰 사용량 라우터(daemon/src/routes/usage.rs)에 있으며, 사용량 기록(POST)·조회(GET)·플랜 단위 집계와 한 모듈을 이룬다. 예산은 봉투 상속 체인을 deep-merge로 해석한 값을 사용한다.

## 미확정 (OPEN)
- [ ] OPEN: 통과/차단을 가르는 비교 기준이 "초과(>)" 인지 "도달(>=)" 인지, 그리고 가드가 차단 응답을 받았을 때의 사용자 안내 흐름(예산 상향 절차)은 운영 정책으로 확정 필요.
