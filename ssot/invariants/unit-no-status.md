---
id: invariant.unit-no-status
kind: Invariant
title: 유닛은 상태도 승인도 없는 순수 그룹핑 엔티티
definition: "유닛은 플랜 안에서 태스크를 묶기만 하는 그룹핑 단위로, 자체 라이프사이클(상태)이나 승인 절차를 갖지 않는다."
governs:
  - concept.unit
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
implementedIn:
  - daemon/src/repo/units.rs
  - daemon/migrations/013_drop_unit_status_approval.sql
  - clawket/prompts/shared/rules.md
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

유닛은 플랜 안에서 여러 태스크를 하나의 작업 그룹으로 묶는 역할만 한다. 플랜처럼 초안→활성→완료로 흐르는 상태도 없고, 승인 게이트도 없다. 즉 유닛은 "진행 중이다 / 끝났다"는 라이프사이클을 스스로 갖지 않는다 — 그저 묶음(bucket)일 뿐이다. 진행 상태를 갖는 것은 그 안의 사이클과 태스크다.

## 깨지면 무슨 일이 일어나나

유닛에 별도의 상태·승인을 부여하면 작업 계층에 군더더기 단계가 생긴다. 플랜이 승인 의도를 담고, 사이클이 진행 타임박스를 담고, 태스크가 실제 작업을 담는 명확한 역할 분담이 흐려지고, "유닛도 승인해야 하나?"라는 불필요한 게이트가 워크플로우에 끼어든다. 유닛을 순수 그룹핑으로 묶어두면 계층의 책임 경계가 단순하게 유지된다.

## 코드에서 어떻게 강제되나

데몬 측 유닛 저장소는 유닛을 순수 그룹핑 엔티티로 다루며 상태·승인 필드를 두지 않는다(`daemon/src/repo/units.rs`). 과거에 존재했을 수 있는 유닛 상태/승인 컬럼은 데이터베이스 마이그레이션으로 제거되었다(`daemon/migrations/013_drop_unit_status_approval.sql`이 유닛의 status·approval을 드롭). 다만 유닛은 자신이 속한 플랜이 완료된 경우에는 그 아래에 새 유닛을 만들 수 없도록 막혀 있는데, 이는 유닛 자체의 상태가 아니라 상위 플랜의 상태에 따른 제약이다. 워크플로우 규약 문서에서도 "Units group tasks. Tasks are the only entity worked directly."로 유닛의 그룹핑 성격을 명시한다(`clawket/prompts/shared/rules.md`).

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 유닛에서 상태/승인을 제거하기로 한 결정 노드 미확인
