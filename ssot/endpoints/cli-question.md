---
id: endpoint.cli-question
kind: Endpoint
title: clawket question (create/answer/view/list)
definition: 사람의 명확화·결정을 요청하는 질문을 만들고 답하고 조회하는 CLI 명령군.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.question
    type: mutates
    note: 질문 생성·답변의 주 진입점
  - to: concept.task
    type: backed-by
    note: 질문은 plan/unit/task에 첨부
  - to: concept.audit-log
    type: relates-to
    note: 질문·답변이 감사 로그에 기록됨
governedBy: []
impacts:
  - domain.governance-audit
consumedBy:
  - component.skills
  - component.claude-adapter
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
질문(question)은 에이전트가 혼자 결정할 수 없는 지점에서 사람에게 명확화나 의사결정을 요청하는 엔티티다. "이 요구사항의 의도가 무엇인가", "두 방향 중 어느 쪽으로 갈까", "외부 요인으로 막혔다" 같은 물음을 작업 맥락(플랜·유닛·태스크) 안에 기록해, 답이 올 때까지 묻고 답한 내용이 작업 흐름에 그대로 남도록 한다.

## 요청 / 응답
- **만들기(create)**: 질문 본문과 함께 어디에 붙일지(플랜·유닛·태스크), 질문 종류(명확화·결정·차단요인), 어디서 비롯됐는지(프롬프트·플랜·리뷰), 누가 물었는지를 받아 미답 질문을 연다.
- **답변(answer)**: 열린 질문에 답변 텍스트를 달고, 답한 주체(사람·main·특정 에이전트)를 기록한다.
- **보기(view)**: 단건 질문 상세를 돌려준다.
- **목록(list)**: 플랜·유닛·태스크 기준으로 거르며, 미답 질문만 보기 옵션을 지원한다.

## 권한 / 제약
- 질문은 플랜·유닛·태스크 중 하나에 첨부되어 작업 맥락을 유지한다.
- 질문 종류·출처는 기본값(명확화·프롬프트)을 가지며, 답변 주체는 기본적으로 사람(human)으로 기록된다.

## provenance
- 명령·옵션 정의: `cli/src/main.rs`의 QuestionAction 정의(create/answer/view/list).
- 실제 저장: clawketd 데몬의 question HTTP API. 미답 질문 필터를 통해 사람의 결정 대기 지점을 추적한다.

## 미확정 (OPEN)
- [ ] OPEN: 미답 질문이 태스크 진행·차단 상태와 연동되어 작업 흐름을 실제로 막는지(블로커 종류와 blocked 상태의 관계) 확인 필요.
