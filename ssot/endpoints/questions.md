---
id: endpoint.questions
kind: Endpoint
title: 질문(question) 조회·생성·답변 API
definition: "웹 대시보드가 사람의 명확화·결정을 요청하는 질문 엔티티를 목록 조회·단건 조회·생성하고, 받은 질문에 답변을 다는 데 쓰는 question 관련 API 묶음."
realizedBy:
  - component.daemon
implementedIn:
  - web/src/api.ts
relatesTo:
  - to: concept.question
    type: reads
    note: 질문 엔티티를 조회·생성·답변한다
  - to: concept.task
    type: reads
    note: 질문은 플랜/유닛/태스크 어느 한 곳에 붙어 그 작업 단위의 맥락을 가진다
consumedBy:
  - component.web-dashboard
governedBy: []
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
에이전트나 사람이 "이 결정은 사람이 해 줘야 한다"는 명확화·결정 요청을 남기고, 사람이 그에 답하는 질문(question) 엔티티를 다루는 API 묶음이다. 웹 대시보드의 태스크 상세에 붙는 질문 패널이 미답변 질문을 보여 주고, 사람이 답을 입력하면 이 API로 답변이 기록된다. 네 가지 동작을 제공한다: 질문 목록 조회, 단건 조회, 질문 생성, 답변 달기.

## 요청 / 응답
- **목록 조회**: 어느 플랜·유닛·태스크에 달린 질문인지, 질문 종류(kind)가 무엇인지로 좁힐 수 있고, "아직 답이 없는 것만" 같은 미답변 필터도 줄 수 있다. 조건에 맞는 질문 목록을 돌려준다.
- **단건 조회**: 질문 식별자 하나로 그 질문의 전체 내용을 돌려준다.
- **생성**: 질문이 어느 작업 단위(플랜/유닛/태스크 중 하나)에 붙는지, 질문의 종류와 출처(origin), 본문, 누가 물었는지를 보내면 새 질문 한 건이 만들어져 돌아온다.
- **답변**: 답할 질문의 식별자와 함께 답변 본문(필수)과 선택적으로 답변자를 보내면, 답변이 반영된 질문이 돌아온다.

각 질문은 의미상 "어느 작업 단위에 속하는가", "종류·출처", "질문 본문", "물은 사람", 그리고 답변이 달렸다면 "답변 본문·답변자"를 담는다.

## 권한 / 제약
- 로컬 데몬을 호출하는 같은 사용자(사람 운영자·LLM 에이전트)가 사용하며, 데몬이 로컬 전용이라 외부 노출이 없다.
- 질문은 반드시 어떤 작업 단위(플랜·유닛·태스크)에 묶이도록 의도되어 있어, 생성 시 그 소속과 함께 누가 물었는지(asked_by)가 채워진다. 답변은 본문이 비어 있을 수 없다.

## provenance
웹 클라이언트 호출부는 `web/src/api.ts`의 질문 관련 함수들(`listQuestions`/`getQuestion`/`createQuestion`/`answerQuestion`)이다. 답변은 질문 식별자에 답변 본문을 실어 보내는 별도 경로(`/questions/:id/answer`)를 쓴다. 실제 저장·처리는 데몬(component.daemon)이 맡으므로, 응답 형태가 바뀌어도 "작업 단위에 묶인 질문을 만들고 답한다"는 의미가 유지되면 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 라우터의 question 처리 정독으로 질문 종류(kind)·출처(origin)의 허용 어휘와 미답변 필터 인자의 정확한 표현을 자연어로 확정할 것.
