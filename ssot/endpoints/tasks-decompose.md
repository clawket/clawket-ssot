---
id: endpoint.tasks-decompose
kind: Endpoint
title: 데몬 POST /tasks/{id}/decompose
definition: 한 태스크를 어떻게 잘게 나눌지 "제안"을 만들어 돌려주는 통로. 실제로 하위 태스크를 만들지는 않고, 분해안만 제시해 사람·에이전트가 골라 수락하게 한다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 대상 태스크와 기존 하위 개수를 읽어 분해안을 만든다
  - to: concept.execution-envelope
    type: reads
    note: 태스크의 실행 봉투(성공기준·분해정책)를 입력으로 삼아 분해안을 생성한다
  - to: component.daemon
    type: backed-by
    note: 분해 제안을 생성하는 로컬 데몬
governedBy: []
impacts:
  - capability.decompose-task
consumedBy:
  - capability.decompose-task
  - endpoint.cli-task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
한 태스크를 어떻게 더 작은 단위로 쪼갤지 "제안서"를 만들어 돌려주는 통로다. 중요한 점은 이 통로가 실제로 하위 태스크를 생성하지 않는다는 것이다 — 분해안(어떤 하위 작업들로 나누면 좋을지)만 계산해 제시하고, 그중 무엇을 실제로 만들지는 사람이나 에이전트가 별도로 수락하는 단계에서 결정한다. 즉 "나눌 거리"를 제안하는 단계와 "실제로 쪼개 만드는" 단계를 분리해, 무분별한 자동 분해를 막는다.

## 요청 / 응답
보내는 정보는 분해 방식을 조절하는 힌트다 — 어떤 전략으로 나눌지(예: 자동/범위기준/레포기준 같은 전략 힌트)와, 제안을 몇 단계 깊이까지 펼칠지(깊이 한도)다. 둘 다 생략 가능하며, 생략하면 기본값(전략은 "자동", 깊이는 2단계, 1~3 범위로 제한)이 적용된다. 응답으로는 제안된 분해안 — 즉 만들면 좋을 하위 작업들의 구조 — 을 돌려준다. 제안을 만들 때는 대상 태스크의 실행 계약(성공 기준·분해 정책 등)과, 이미 달려 있는 하위 작업의 개수를 함께 참고해, 이미 충분히 쪼개진 경우 중복 제안을 줄인다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 호출된다. 대상 태스크가 없으면 "찾을 수 없음"으로 응답한다. 분해 깊이는 1~3 단계로 제한되며, 알 수 없는 전략 값은 "자동"으로 정규화되어 안전하게 처리된다. 이 통로는 읽고 계산만 할 뿐 상태를 바꾸지 않는다 — 실제 하위 태스크 생성은 서브태스크 생성 통로의 몫이다.

## provenance
근거는 `daemon/src/routes/tasks.rs`의 라우트 등록(`/tasks/{id}/decompose`, POST → `decompose_route`)과 핸들러 `decompose_route`다. 요청 힌트는 `strategy`(기본 "auto", 미지값은 자동 정규화)와 `max_depth`(기본 2, 1~3로 clamp)이며, 핸들러는 태스크의 해소된 실행 봉투(`resolve_envelope`)와 기존 하위 개수(`tasks::descendants`로 BFS 카운트)를 입력으로 `decompose_suggest::generate`를 호출해 제안만 반환한다(생성 부수효과 없음). 생성 주체는 로컬 데몬(component.daemon)이며, 제안→수락→실행 흐름은 분해 역량(capability.decompose-task)이 묶는다.

## 미확정 (OPEN)
- [ ] OPEN: 전략별("auto"/"scoped"/"by-repo") 분해 규칙과 제안 항목의 구체 형태(각 제안에 어떤 메타가 붙는지), 그리고 깊이 한도와 기존 하위 개수가 제안 수에 미치는 영향은 `decomposition/suggest.rs`를 정독해 확인 필요.
