---
id: endpoint.comment
kind: Endpoint
title: clawket comment (태스크·유닛·플랜 코멘트 관리)
definition: 태스크·유닛·플랜에 코멘트를 달고 목록·수정·삭제하는 명령 묶음. 작업의 진행 메모와 취소된 작업의 사연을 보존하는 수단으로 쓰인다.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.comment, type: mutates, note: "코멘트를 생성·수정·삭제한다" }
  - { to: concept.comment, type: reads, note: "코멘트 목록 조회" }
  - { to: concept.task, type: reads, note: "태스크에 코멘트를 붙일 수 있다" }
  - { to: concept.plan, type: reads, note: "플랜에 코멘트를 붙일 수 있다" }
  - { to: concept.unit, type: reads, note: "유닛에 코멘트를 붙일 수 있다" }
governedBy: []
impacts:
  - domain.work-management
consumedBy:
  - persona.human-operator
  - persona.llm-coding-agent
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
작업 단위(태스크·유닛·플랜)에 사람이 읽는 메모를 남기는 명령 묶음이다. 진행 상황, 판단 근거, 막힌 이유 같은 맥락을 본문(마크다운)으로 적는다. 특히 작업을 취소할 때 항목을 파괴적으로 지우는 대신 "왜 취소했는지"를 코멘트로 남겨 히스토리를 보존하는 용도로 권장된다 — 기록은 사라지지 않고 사연이 남는다.

세부 동작은 네 갈래다.
- 생성: 본문을 적고 태스크·유닛·플랜 중 하나에 붙인다. 작성자(기본은 메인 에이전트)와 분류용 라벨을 선택적으로 단다.
- 목록: 특정 태스크/유닛/플랜에 달린 코멘트들을 본다.
- 수정: 기존 코멘트의 본문을 새 내용으로 교체한다.
- 삭제: 코멘트를 제거한다.

## 요청 / 응답
- 생성에 보내는 것: 본문(마크다운), 붙일 대상(태스크/유닛/플랜 중 하나), 선택적 작성자와 라벨. 돌려받는 것: 만들어진 코멘트의 식별자.
- 목록에 보내는 것: 걸러낼 대상(태스크/유닛/플랜). 돌려받는 것: 코멘트 목록 — 각 항목은 본문·작성자·라벨·작성 시각의 의미를 갖는다.
- 수정에 보내는 것: 코멘트 식별자와 새 본문. 삭제에 보내는 것: 코멘트 식별자.

## 권한 / 제약
로컬 데몬을 통해 사람·에이전트가 호출한다. 코멘트는 태스크·유닛·플랜 중 하나에 귀속되어야 한다. 작성자 이름은 환경 변수(에이전트 이름)에서 채워질 수 있어, 에이전트가 단 코멘트와 사람이 단 코멘트를 작성자로 구분할 수 있다. 본문은 하이픈으로 시작하는 값도 허용해 마크다운 목록·옵션 인용 같은 내용을 그대로 담을 수 있다.

## provenance
명령 정의는 cli/src/main.rs의 Comment 서브커맨드(상위 라우팅 156-161행, 액션 정의 1290-1338행)에 있고, 코멘트의 영속화는 데몬(component.daemon)이 수행한다. 코드가 사라져도 위 정책(태스크/유닛/플랜 귀속, 취소 히스토리 보존 용도, 작성자 구분, 마크다운 본문)만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 라벨이 어떤 정해진 어휘를 갖는지(자유 텍스트인지 enum인지)와, 라벨이 웹 대시보드·검색에서 어떻게 활용되는지 데몬·웹 측에서 확인.
- [ ] OPEN: 코멘트 삭제가 실제 물리 삭제인지 감사 로그에 흔적을 남기는지(concept.audit-log) 대조해 보강.
