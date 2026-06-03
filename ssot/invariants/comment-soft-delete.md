---
id: invariant.comment-soft-delete
kind: Invariant
title: 코멘트 삭제는 행을 지우지 않고 [DELETED] 표시만 남긴다
definition: 코멘트를 삭제해도 기록 자체는 사라지지 않고 본문 앞에 [DELETED] 표시가 붙어 남으며, 화면은 이 표시를 감지해 회색으로 흐려 보여 대화 스레드의 연속성을 보존한다.
governs:
  - concept.comment
  - concept.task
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
  - desktop/apps/desktop/src/data/types.ts
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 자연어 우선. 식별자는 provenance/근거에만. -->

## 제약

코멘트를 "삭제"하더라도 그 코멘트 기록은 데이터에서 영구히 사라지지 않는다. 대신 본문 맨 앞에 삭제됨을 뜻하는 표시가 덧붙은 채로 그 자리에 남는다. 화면은 이 표시를 알아채서 해당 코멘트를 회색으로 흐리게 보여, 사용자가 "여기 코멘트가 있었고 지워졌다"는 사실을 그대로 인지하게 한다. 이는 파괴적 삭제가 아니라 흔적을 남기는 부드러운 삭제(soft delete)이며, Clawket이 취소·삭제 시 히스토리를 보존하려는 일관된 태도와 같은 선상에 있다.

## 깨지면 무슨 일이 일어나나

코멘트를 행 단위로 완전히 지운다면, 그 코멘트가 받았던 답글·그 코멘트를 전제로 이어진 대화의 맥락이 끊긴다. 스레드를 나중에 읽는 사람은 빈 구멍 위에서 앞뒤가 맞지 않는 대화를 보게 되고, "왜 이런 결정을 했는지"의 단서가 통째로 증발한다. 감사 추적과 의사결정 복원이라는 Clawket의 목적상, 코멘트가 흔적 없이 사라지는 것은 작업 히스토리의 단일 진실을 훼손한다. 반대로 표시만 붙여 남기면, 삭제 의도는 화면에 반영하면서도 스레드의 구조와 맥락은 깨지지 않는다.

## 코드에서 어떻게 강제되나

데스크탑 데몬 클라이언트는 삭제 시 데몬이 본문 앞에 삭제 표시를 붙이고 행은 테이블에 남긴다는 동작을 명시하며, 화면 렌더러가 이 표시를 떼어내고 회색 처리해야 한다고 못 박는다(`desktop/apps/desktop/src/data/api.ts`). 표시 문자열은 상수로 정의되어 클라이언트와 화면이 같은 기준으로 감지하도록 공유되고, 코멘트 타입 정의에도 이 약속이 반영되어 있다(`desktop/apps/desktop/src/data/types.ts`). 삭제 호출의 응답 형식은 위로 노출하지 않고 목록을 다시 불러오는 방식으로 화면을 갱신한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 코멘트 부드러운 삭제(흔적 보존) 정책 결정을 decision 노드로 연결.
- [ ] OPEN: 데몬 측에서 표시 접두를 붙이는 실제 강제 위치를 데몬 소스에서 추가 확인.
