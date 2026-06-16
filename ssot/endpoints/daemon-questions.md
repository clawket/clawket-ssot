---
id: endpoint.daemon-questions
kind: Endpoint
title: 데몬 질문 API (/questions, /questions/:id/answer)
definition: "사람의 명확화·결정을 요청하는 질문을 다루는 데몬 엔드포인트. 질문 목록 조회와 생성(/questions), 그리고 답변 등록(POST /questions/:id/answer)을 제공하며, 작업 단위로 필터링하거나 미답변(pending)만 추릴 수 있다."
realizedBy: [component.daemon]
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - { to: concept.question, type: mutates, note: "질문 생성과 답변 등록을 다룬다" }
  - { to: concept.task, type: reads, note: "질문은 보통 특정 작업에 종속되어 그 작업의 명확화를 요청한다" }
  - { to: persona.human-operator, type: relates-to, note: "에이전트가 던진 질문에 사람이 답하는 핸드오프 지점이다" }
impacts: [concept.question]
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

에이전트가 작업을 진행하다 사람의 판단이 필요할 때(요구사항이 모호하거나, 도메인 의미·우선순위 결정이 필요할 때) 질문을 남기고, 사람이 그에 답하는 핸드오프 채널이다. 질문은 작업과 사람 사이의 비동기 결정 통로 역할을 한다.

세 가지 동작을 제공한다 — 질문 목록 조회, 질문 생성, 그리고 특정 질문에 대한 답변 등록이다.

## 요청 / 응답

- **목록**: 질문들을 돌려준다. 특정 작업으로 범위를 좁히거나, 아직 답이 달리지 않은 미답변 질문만 추려 받을 수 있다. 각 항목은 본문·질문 유형·출처·질문자·답변 여부 등을 담는다.
- **생성**: 어느 작업에 대한 질문인지와 질문 본문을 보낸다. 질문 유형·출처를 선택적으로 덧붙일 수 있고, 질문자를 명시하지 않으면 기본값(에이전트 측 "main")으로 귀속된다.
- **답변**: 특정 질문에 답변 본문을 등록한다. 답변자를 함께 명시할 수 있다. 답변이 등록되면 그 질문은 더 이상 미답변이 아니게 된다.

## 권한 / 제약

- 질문은 작업·유닛·플랜에 붙을 수 있으나, 데스크탑·웹 화면은 작업별 질문 흐름만 소비한다.
- 로컬 데몬 특성상 호출 주체는 같은 머신의 사람·에이전트로 한정된다.

## provenance

- implementedIn: 데스크탑 데몬 클라이언트가 목록 필터(작업·미답변), 생성 필수/선택 필드와 기본 질문자, 답변 등록 경로를 주석과 함께 노출한다.
- realizedBy: clawketd 데몬이 질문 저장소를 소유하고 이 API를 제공한다.

## 미확정 (OPEN)
- [ ] OPEN: 답변 등록이 연결된 작업의 blocked 해제 등 다른 상태 변화를 트리거하는지 여부.
