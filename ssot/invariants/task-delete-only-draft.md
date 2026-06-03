---
id: invariant.task-delete-only-draft
kind: Invariant
title: 태스크 완전 삭제는 플랜이 draft일 때만 허용된다
definition: 태스크를 흔적 없이 완전히 지우는 것은 그 태스크가 아직 시작 전(todo)이고 소속 플랜이 draft 상태일 때만 가능하며, 그 외에는 삭제 요청이 취소(cancelled) 처리로 바뀌어 히스토리가 보존된다.
governs:
  - concept.task
  - concept.plan
  - capability.manage-task-lifecycle
  - domain.work-management
implementedIn:
  - daemon/src/routes/tasks.rs
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

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 제약

태스크를 데이터베이스에서 흔적 없이 완전히 지우는 "하드 삭제"는, 다음 두 조건이 모두 충족될 때만 허용된다.

1. 그 태스크가 아직 시작되지 않은 상태(todo)일 것.
2. 그 태스크가 속한 플랜이 아직 확정 전, 즉 draft 상태일 것.

이 두 조건이 모두 참이면 태스크는 깨끗이 삭제된다. 하나라도 어긋나면 — 예를 들어 태스크가 이미 진행 중이거나, 플랜이 이미 승인되어 active가 되었거나 — 삭제 요청은 거부되는 대신 "취소(cancelled)"로 전환되고, 취소 사유가 시스템 코멘트로 붙어 작업 흔적이 남는다.

## 깨지면 무슨 일이 일어나나

만약 어떤 상태의 태스크든 자유롭게 완전 삭제할 수 있다면, 이미 진행했거나 승인된 계획의 일부였던 작업이 아무 기록도 없이 사라질 수 있다. 그러면 "왜 이 작업이 사라졌는지", "원래 계획에 무엇이 있었는지"를 추적할 수 없고, 감사 추적과 작업 히스토리가 구멍이 난다. 이는 Clawket의 핵심 가치인 "모든 작업 이력을 영구 보존한다"는 원칙(별도 불변식 invariant.preserve-history)과 정면으로 충돌한다.

규칙이 작동하면, 진짜로 아무 의미 없는 초안 단계의 잘못 만든 태스크만 깨끗이 지울 수 있고, 그 외의 모든 "삭제"는 안전하게 취소-보존으로 흡수된다.

## 코드에서 어떻게 강제되나

태스크 삭제 요청을 처리하는 데몬 라우트(`daemon/src/routes/tasks.rs`의 삭제 핸들러)에서, 먼저 태스크 상태가 todo인지 확인하고, 그렇다면 그 태스크의 유닛을 거쳐 플랜을 조회해 플랜이 draft인지 본다. 둘 다 참이면 실제 삭제를 수행하고 "삭제됨" 이벤트를 알린다. 조건이 어긋나면 같은 핸들러가 곧바로 소프트 삭제 경로로 빠져, 상태를 cancelled로 바꾸고 `[Cancelled] …` 시스템 코멘트를 붙여 감사 흔적을 남긴다(코드 주석상 Node v2.2.1과의 동작 호환을 명시).

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 규칙이 도입된 결정 기록) 연결 필요.
