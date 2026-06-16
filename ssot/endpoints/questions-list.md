---
id: endpoint.questions-list
kind: Endpoint
title: 데몬 GET/POST /questions
definition: 사람의 명확화·결정 요청(질문)을 목록으로 조회하거나 새로 등록하는 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/questions.rs]
relatesTo:
  - { to: concept.question, type: reads, note: 질문 목록을 조회한다 }
  - { to: concept.question, type: mutates, note: 새 질문을 생성한다 }
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
에이전트가 작업 중 사람의 판단이 필요할 때(명확화·결정 요청) 남기는 "질문"을 다루는 두 동작을 한 주소에 묶은 엔드포인트다. 같은 경로에서 목록을 읽거나, 새 질문을 등록할 수 있다. 목록 조회는 어떤 작업(플랜·유닛·태스크)에 달린 질문인지, 아직 답이 없는 것만 볼지로 좁힐 수 있다. CLI의 질문 명령이 이 기능을 사용자에게 노출한다.

## 요청 / 응답
목록 조회 시 보내는 정보(모두 선택): 어느 플랜·유닛·태스크에 속한 질문으로 범위를 좁힐지, 그리고 "아직 답변되지 않은 질문만" 추리는 조건. 돌려주는 정보: 조건에 맞는 질문 목록 — 각 질문은 어떤 작업에 달렸는지, 질문 종류, 어디서 비롯됐는지(예: 프롬프트), 본문, 누가 물었는지, 답변 상태 등을 가진다.

새 질문 등록 시 보내는 정보: 질문이 속할 작업(플랜·유닛·태스크 중 하나), 질문 본문(필수), 그리고 선택적으로 질문 종류·출처·물은 주체. 종류와 출처를 생략하면 각각 "명확화 요청"과 "프롬프트에서 비롯됨"으로 기본 처리된다. 돌려주는 정보: 막 만들어진 질문 한 건.

## 권한 / 제약
로컬 데몬에 붙은 호출자가 사용하는 엔드포인트다. 새 질문에는 본문이 반드시 있어야 하고, 질문 종류·출처를 비우면 기본값으로 채워진다.

## provenance
구현은 데몬 라우터의 questions 모듈(`daemon/src/routes/questions.rs`)로, `/questions` 경로에 목록(GET)과 생성(POST)을 함께 등록한다. 제공 주체는 로컬 clawketd 데몬(`component.daemon`)이다. 위 요청/응답 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 질문 "종류" 값의 표준 어휘 집합과 "출처" 값의 의미 분류는 모델·UI를 함께 확인해 owner 가 확정할 것.
