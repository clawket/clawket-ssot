---
id: concept.comment
kind: Concept
title: Comment
definition: "태스크에 붙는 사람·에이전트의 메모(코멘트). 작성자·본문·작성 시각을 남기며 작성 순서대로 보존된다. 단순한 토막글이 아니라, '파괴적으로 지우는 대신 코멘트로 맥락을 남긴다'는 운영 원칙의 핵심 수단이다 — 취소된 작업의 사유나 이력을 코멘트로 남겨 모든 작업 이력이 영구 보존되도록 한다. 코멘트 자체도 하드 삭제 대신 소프트 삭제(본문 앞에 [DELETED] 표식을 붙여 흔적을 남김)가 기본이다."
relatesTo:
  - to: concept.task
    type: relates-to
    note: 코멘트는 태스크에 붙는다(현 스키마는 태스크 코멘트만 지원)
  - to: endpoint.cli-comment
    type: reads
    note: clawket comment create/list/update/delete 명령이 코멘트를 다룬다
  - to: concept.audit-log
    type: relates-to
    note: 둘 다 '파괴적 삭제 대신 이력 보존'이라는 같은 원칙을 구현한다(코멘트=맥락 보존, 감사 로그=변경 추적)
governedBy: []
governs: []
dependsOn: []
implementedIn:
  - daemon/src/repo/comments.rs
  - daemon/src/routes/comments.rs
  - daemon/migrations/001_initial.sql
  - clawket/prompts/shared/rules.md
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

코멘트는 태스크에 덧붙이는 짧은 메모다. 누가 썼는지(작성자), 무슨 내용인지(본문), 언제 썼는지(작성 시각)를 담고, 한 태스크에 달린 코멘트들은 작성한 순서대로 쌓여 시간순으로 읽힌다.

코멘트는 단순한 부가 메모를 넘어 Clawket의 운영 철학을 구현하는 수단이다. Clawket에는 "**작업을 파괴적으로 삭제하지 말고, 취소된 작업은 코멘트로 이력을 보존하라**"는 원칙이 있다(작업 규칙 문서에 명시). 예를 들어 어떤 태스크를 중단·취소할 때 그 태스크를 지워 버리는 대신, 취소 사유와 맥락을 코멘트로 남겨 둔다. 이렇게 하면 "왜 이 작업이 멈췄는가"가 영구히 남아, 모든 작업 이력이 로컬에 보존된다는 약속(작업 규칙: 모든 작업 이력은 로컬에 영구 저장)이 지켜진다.

이 "지우지 말고 흔적을 남긴다"는 정신은 코멘트 자체의 삭제에도 적용된다. 코멘트를 지우라는 요청이 와도 기본 동작은 행을 통째로 없애는 것이 아니라, 본문 앞에 [DELETED] 표식을 붙여 "삭제된 코멘트가 여기 있었다"는 흔적을 남기는 소프트 삭제다.

## 엔티티 (DB)

코멘트는 task_comments 테이블에 한 건씩 저장된다. 한 행이 담는 정보:

- 코멘트 식별자(접두사 CMT가 붙은 시간정렬 식별자).
- 소속 태스크(태스크가 삭제되면 그 코멘트들도 함께 사라진다 — 종속 삭제).
- 작성자, 본문(둘 다 필수), 작성 시각.

태스크별 조회를 위한 색인이 있다. 현재 스키마는 **태스크 코멘트만** 가진다 — 유닛/플랜에 직접 붙는 코멘트 테이블은 없다.

소프트 삭제는 별도 컬럼 없이 본문 텍스트 앞에 [DELETED] 접두사를 붙이는 방식으로 구현된다. 하드 삭제(행 제거) 함수도 코드에는 존재하지만 일반 API 경로에서는 쓰이지 않는다.

## API 표면

데몬 HTTP 표면:

- 태스크별 코멘트 목록 조회 / 태스크에 코멘트 추가(작성 순서대로 반환).
- 컬렉션 경로(/comments): CLI 호환을 위해 본문에 태스크 식별자를 담아 추가/조회할 수 있다. 단 **유닛/플랜 식별자를 넣으면 거부**된다("유닛/플랜 코멘트는 지원하지 않음 — 스키마에 태스크 코멘트만 있음"). 즉 도구 인터페이스 차원에서는 유닛/플랜 코멘트라는 표현이 남아 있으나 실제로는 태스크에만 붙는다.
- 코멘트 본문 수정(PATCH).
- 코멘트 삭제(DELETE): 하드 삭제가 아니라 **소프트 삭제** — 본문 앞에 [DELETED]를 붙이고 "soft_deleted" 결과를 돌려준다.

CLI에서는 clawket comment의 create/list/update/delete로 같은 일을 한다.

## 불변식

- **태스크 종속**: 모든 코멘트는 하나의 태스크에 속하며, 태스크가 삭제되면 함께 삭제된다.
- **태스크 전용**: 현 스키마는 태스크 코멘트만 지원한다. 유닛/플랜 코멘트 생성 요청은 거부된다.
- **소프트 삭제 우선**: 삭제 API는 행을 제거하지 않고 [DELETED] 표식을 붙여 흔적을 보존한다(이력 보존 원칙).
- **순서 보존**: 코멘트는 작성 시각 오름차순으로 보존·반환된다.

## 구현 위치 (provenance)

- 코멘트 생성·조회·목록·본문 수정·소프트 삭제: `daemon/src/repo/comments.rs`.
- HTTP 라우트(태스크별/컬렉션 추가·조회, PATCH 수정, DELETE 소프트 삭제, 유닛·플랜 거부): `daemon/src/routes/comments.rs`.
- task_comments 테이블 정의: `daemon/migrations/001_initial.sql`.
- "파괴적 삭제 대신 코멘트로 이력 보존" 운영 원칙: `clawket/prompts/shared/rules.md`.

## 미확정 (OPEN)
- [ ] OPEN: 태스크를 취소(cancelled)할 때 취소 사유 코멘트를 자동으로 남기는 강제 장치가 있는지(규칙은 권고로 보이며, 자동 기입 경로는 미확인).
- [ ] OPEN: 노드 소스의 `src/main.rs:1290-1338`는 CLI 레포 기준 경로 — CLI(`clawket comment`)의 정확한 구현 위치는 별도 추적 필요(본 노드는 데몬 측 코멘트 저장·라우트만 확정).
