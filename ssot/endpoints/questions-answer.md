---
id: endpoint.questions-answer
kind: Endpoint
title: 데몬 POST /questions/{id}/answer
definition: 미해결 질문에 사람의 답변을 등록해 결정·명확화를 확정하는 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/questions.rs]
relatesTo:
  - { to: concept.question, type: mutates, note: 질문에 답변과 답변 주체를 채워 해결 상태로 전환한다 }
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
에이전트가 남긴 질문에 사람이 답을 달아 결정을 확정하는 기능이다. 답변이 등록되면 그 질문은 "해결됨" 상태가 되어, 미해결 질문만 추리는 목록 조회에서 빠진다. 작업을 막고 있던 명확화 요청을 사람이 처리하는 지점이다.

## 요청 / 응답
보내는 정보: 답변할 질문의 식별자, 답변 본문(필수), 그리고 선택적으로 답변한 주체. 돌려주는 정보: 답변이 반영된 질문 한 건 — 답변 본문·답변 주체와 함께 해결 상태로 갱신된 모습. 해당 식별자의 질문이 없으면 "없음(404)"으로 응답한다.

## 권한 / 제약
로컬 데몬에 붙은 호출자(통상 사람 운영자)가 사용한다. 답변 본문은 반드시 있어야 한다. 답변 주체는 생략 가능하다.

## provenance
구현은 데몬 라우터의 questions 모듈(`daemon/src/routes/questions.rs`)로, `/questions/{id}/answer` 경로의 POST으로 등록된다. 제공 주체는 로컬 clawketd 데몬(`component.daemon`)이다. "질문에 답을 달면 해결 상태로 바뀐다"는 정책과 위 요청/응답 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 이미 답변된 질문에 다시 답변할 때의 동작(덮어쓰기 허용 여부)은 저장소 계층(questions repo)을 확인해 owner 가 확정할 것.
