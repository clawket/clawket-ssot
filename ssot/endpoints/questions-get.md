---
id: endpoint.questions-get
kind: Endpoint
title: 데몬 GET /questions/{id}
definition: 식별자로 단일 질문 한 건을 조회하는 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/questions.rs]
relatesTo:
  - { to: concept.question, type: reads, note: 식별자로 질문 한 건을 읽는다 }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-question]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
이미 알고 있는 질문 식별자로 그 질문 한 건의 전체 내용을 가져오는 단순 조회 기능이다. 목록에서 본 질문의 상세를 확인하거나, 답변 전후의 상태를 확인할 때 쓴다.

## 요청 / 응답
보내는 정보: 조회할 질문의 식별자. 돌려주는 정보: 해당 질문 한 건 — 어떤 작업(플랜·유닛·태스크)에 달렸는지, 질문 종류, 출처, 본문, 물은 주체, 답변 본문과 답변 주체, 답변 여부 등. 해당 식별자의 질문이 없으면 "없음(404)"으로 응답한다.

## 권한 / 제약
로컬 데몬에 붙은 호출자가 사용하는 조회성 엔드포인트로, 상태를 바꾸지 않는다. 존재하지 않는 식별자에는 404를 돌려준다.

## provenance
구현은 데몬 라우터의 questions 모듈(`daemon/src/routes/questions.rs`)로, `/questions/{id}` 경로의 GET으로 등록된다. 제공 주체는 로컬 clawketd 데몬(`component.daemon`)이다. 위 요청/응답 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 응답에 포함되는 질문 필드의 완전한 목록·의미는 질문 모델 정의를 확인해 owner 가 확정할 것.
