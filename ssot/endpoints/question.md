---
id: endpoint.question
kind: Endpoint
title: clawket question (사람 명확화·결정 질문 관리)
definition: 사람의 명확화·결정·차단 해소를 요청하는 질문을 만들고 답하고 조회·목록하는 명령 묶음. 에이전트가 혼자 판단할 수 없는 지점을 사람에게 올리는 통로.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.question, type: mutates, note: "질문을 생성하고 답변을 기록한다" }
  - { to: concept.question, type: reads, note: "질문 조회·목록" }
  - { to: concept.task, type: reads, note: "질문을 태스크에 붙일 수 있다" }
  - { to: concept.plan, type: reads, note: "질문을 플랜에 붙일 수 있다" }
  - { to: concept.unit, type: reads, note: "질문을 유닛에 붙일 수 있다" }
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
LLM 에이전트가 혼자 결정할 수 없거나 사람의 명시적 판단이 필요한 지점을, 작업 맥락에 묶어 사람에게 공식적으로 올리는 명령 묶음이다. 단순한 채팅 메시지가 아니라 "답이 달리기를 기다리는 미해결 질문"이라는 상태를 갖고, 답이 달리면 누가 언제 무엇이라 답했는지가 함께 보존된다. 질문은 성격에 따라 명확화 요청·결정 요청·차단 해소 요청으로 구분된다.

세부 동작은 네 갈래다.
- 생성: 질문 본문을 적고 태스크·유닛·플랜 중 하나에 붙인다. 질문의 성격(명확화/결정/차단)과 발생 출처(프롬프트/플랜/리뷰), 질문한 주체를 함께 남긴다.
- 답변: 미해결 질문에 답을 단다. 누가 답했는지(사람/메인/특정 에이전트)를 기록한다.
- 조회·목록: 한 질문을 펼쳐 보거나, 특정 태스크/유닛/플랜으로 걸러 목록을 보며, 미답변만 추려 볼 수 있다.

## 요청 / 응답
- 생성에 보내는 것: 질문 본문, 붙일 대상(태스크/유닛/플랜), 질문 성격(명확화/결정/차단), 출처(프롬프트/플랜/리뷰), 질문한 주체. 돌려받는 것: 만들어진 질문의 식별자와 미해결 상태.
- 답변에 보내는 것: 질문 식별자, 답변 텍스트, 답한 주체. 돌려받는 것: 해결 처리된 질문.
- 목록에 보내는 것: 걸러낼 대상과 미답변 한정 여부. 돌려받는 것: 질문 목록 — 각 항목은 본문·성격·출처·질문자·답변 여부·답변 내용의 의미를 갖는다.

## 권한 / 제약
로컬 데몬을 통해 동작한다. 질문은 태스크·유닛·플랜 중 하나에 귀속된다. 질문 성격의 기본값은 명확화, 출처 기본값은 프롬프트다. 질문한 주체는 환경 변수(에이전트 이름)에서, 답한 주체는 기본적으로 사람으로 채워져 — 에이전트가 올리고 사람이 답하는 전형적 흐름을 자연스럽게 표현한다. 답변 텍스트도 하이픈으로 시작하는 값을 허용한다.

## provenance
명령 정의는 cli/src/main.rs의 Question 서브커맨드(상위 라우팅 162-167행, 액션 정의 1231-1288행)에 있고, 질문·답변의 영속화는 데몬(component.daemon)이 수행한다. 코드가 사라져도 위 정책(태스크/유닛/플랜 귀속, 명확화/결정/차단 성격, 출처 분류, 질문자·답변자 주체 구분, 미답변 필터)만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 차단(blocker) 성격 질문이 연결된 태스크의 blocked 상태 전환과 자동 연동되는지(질문 생성→태스크 자동 차단) 데몬·훅 처리에서 확인.
- [ ] OPEN: 미답변 질문이 웹 대시보드·UserPromptSubmit 훅에서 알림/주의 환기로 노출되는지 대조해 보강.
