---
id: concept.question
kind: Concept
title: Question
definition: "에이전트나 시스템이 사람의 명확화·결정을 요청해 기록으로 남기는 질문 엔티티. 플랜·유닛·태스크 중 적어도 하나에 매달려야 하며, 질문의 종류(명확화/결정/블로커/리뷰)와 어디서 비롯됐는지(프롬프트/웹/훅)를 함께 남긴다. 답변이 달리기 전까지는 '대기(pending)' 상태로, 누가 언제 무엇이라 답했는지가 기록되면 해소된다. '에이전트가 막혔을 때 멋대로 진행하지 말고 사람에게 물어 결정을 남긴다'는 협업 원칙을 데이터로 구현한 것이다."
relatesTo:
  - to: concept.task
    type: relates-to
    note: 질문은 태스크에 매달릴 수 있다(플랜/유닛/태스크 중 하나 이상 필수)
  - to: concept.plan
    type: relates-to
    note: 질문은 플랜에도 매달릴 수 있다
  - to: concept.unit
    type: relates-to
    note: 질문은 유닛에도 매달릴 수 있다
  - to: endpoint.cli-question
    type: reads
    note: clawket question create/answer/view/list 명령이 질문을 다룬다
  - to: persona.human-operator
    type: relates-to
    note: 질문은 사람 운영자의 명확화·결정을 요청하는 통로다
governedBy: []
governs: []
dependsOn: []
implementedIn:
  - daemon/src/repo/questions.rs
  - daemon/src/routes/questions.rs
  - daemon/migrations/001_initial.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 자연어 우선. 코드 식별자는 provenance/근거에만. -->

## 정의

질문(Question)은 "여기서 사람의 판단이 필요하다"를 기록으로 남기는 객체다. 에이전트가 작업 중 모호한 점을 만나거나, 사람만이 정할 수 있는 결정(도메인 의미·비즈니스 우선순위 등)에 부딪히거나, 외부 의존으로 막혔을 때, 멋대로 추측해 진행하지 않고 질문을 만들어 사람에게 넘긴다. 사람이 답을 적으면 질문이 해소되고, 그 답은 영구 기록으로 남아 나중에 "왜 이렇게 결정했나"를 되짚을 수 있다.

한 건의 질문이 답하는 것:

- **무엇에 대한 질문인가**: 플랜·유닛·태스크 중 적어도 하나에 반드시 매달려야 한다. 어디에도 걸리지 않은 질문은 만들 수 없다.
- **어떤 종류인가(kind)**: 명확화(clarification) / 결정(decision) / 블로커(blocker) / 리뷰(review). 지정하지 않으면 명확화로 기본 처리.
- **어디서 비롯됐나(origin)**: 프롬프트(prompt) / 웹(web) / 훅(hook). 지정하지 않으면 프롬프트로 기본 처리.
- **누가 물었나(asked_by)**: 메인 에이전트·사람·특정 스킬 등(선택).
- **본문**: 질문 내용.
- **답변과 답한 사람·답한 시각**: 답이 달리기 전까지는 비어 있다. 답이 없는 동안 질문은 "대기(pending)" 상태로 간주된다.

질문은 에이전트와 사람이 같은 상태를 보며 협업하는 Clawket 모델에서, "막힘"을 침묵하거나 임의 진행으로 덮지 않고 명시적 기록으로 드러내는 장치다.

## 엔티티 (DB)

질문은 questions 테이블에 한 건씩 저장된다. 한 행이 담는 정보:

- 질문 식별자(접두사 Q가 붙은 시간정렬 식별자).
- 매달린 대상: 플랜·유닛·태스크 식별자(셋 다 선택이지만 **적어도 하나는 반드시** 있어야 한다 — DB CHECK 제약으로 강제). 대상이 삭제되면 질문도 함께 삭제된다(종속 삭제).
- 종류(kind): 명확화/결정/블로커/리뷰.
- 출처(origin): 프롬프트/웹/훅.
- 본문(필수), 물은 사람(선택), 생성 시각.
- 답변, 답한 사람, 답한 시각(답하기 전엔 모두 비어 있음).

대상별(플랜·유닛·태스크) 색인과, 아직 답이 없는 대기 질문만 빠르게 찾는 색인(답한 시각이 비어 있는 행 대상)이 있다.

## API 표면

데몬 HTTP 표면:

- 목록 조회: 플랜·유닛·태스크로 거르고, "대기 중(답 없음)" 여부로도 걸러 생성 시각 내림차순(최신 먼저)으로 반환한다.
- 생성: 대상(플랜/유닛/태스크)·종류·출처·본문·물은 사람을 받아 질문을 만든다. 종류·출처를 안 주면 각각 명확화·프롬프트가 기본값. 대상이 하나도 없으면 거부.
- 단건 조회.
- 답변: 질문에 답을 적는다. 답한 사람을 안 주면 사람(human)으로 기본 기록하고, 답한 시각을 찍는다.

CLI에서는 clawket question의 create/answer/view/list로 같은 일을 한다.

## 불변식

- **대상 필수**: 모든 질문은 플랜·유닛·태스크 중 적어도 하나에 매달려야 한다(DB CHECK). 무소속 질문은 거부된다.
- **종속 삭제**: 매달린 대상이 삭제되면 질문도 함께 삭제된다.
- **대기/해소 구분**: 답한 시각이 비어 있으면 대기(pending), 채워지면 해소된 것으로 본다. 대기 질문은 전용 색인으로 빠르게 조회된다.

## 구현 위치 (provenance)

- 질문 생성(대상 필수 검사)·조회·목록(대상·대기 필터)·답변 기록: `daemon/src/repo/questions.rs`.
- HTTP 라우트(목록/생성/단건/답변)와 종류·출처 기본값 처리: `daemon/src/routes/questions.rs`.
- questions 테이블 정의(대상 CHECK 제약, 종류·출처 허용값 주석, 대기 색인): `daemon/migrations/001_initial.sql`.

## 미확정 (OPEN)
- [ ] OPEN: 종류(kind)·출처(origin) 값이 DB CHECK로 폐집합 강제되는지(주석에는 명시되나 CHECK 제약 여부는 본 노드에서 미확정 — 코멘트성 주석만 확인) 추가 확인 필요.
- [ ] OPEN: 노드 소스의 `src/main.rs:1231-1288`·`src/types.ts`·`src/api.ts`는 CLI/웹 레포 기준 경로 — CLI(`clawket question`)와 웹의 질문 패널(TaskQuestionsPanel) 구현 위치는 별도 추적 필요.
- [ ] OPEN: 훅 출처(origin=hook) 질문이 실제로 어떤 훅 이벤트에서 생성되는지(에이전트 막힘 감지와의 연결) 미확인.
