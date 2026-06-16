---
id: invariant.task-delete-draft-only
kind: Invariant
title: 태스크 삭제는 플랜이 draft일 때만
definition: 태스크는 그것이 속한 플랜이 아직 draft(초안) 상태일 때만 삭제할 수 있다. 플랜이 한 번 active 이상으로 진행되면 태스크는 더 이상 삭제되지 않는다.
governs:
  - concept.task
  - concept.plan
governedBy: []
implementedIn:
  - cli/src/main.rs
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 제약
태스크의 영구 삭제는 그 태스크가 속한 플랜이 아직 **draft(초안)** 단계일 때로만 허용된다. 플랜이 승인되어 active가 되면, 그 안의 태스크는 더 이상 지워지지 않는다 — 이미 진행에 들어간 작업 단위는 파괴적으로 사라지지 않고 취소(cancelled) 상태나 코멘트로 히스토리를 보존하는 방식으로 다뤄야 한다.

이는 "초안 단계의 계획은 자유롭게 재구성하되, 일단 승인되어 진행이 시작된 작업의 기록은 보존한다"는 원칙을 구현한다.

## 깨지면 무슨 일이 일어나나
이 제약이 없으면 이미 진행 중이거나 완료된 태스크가 통째로 삭제될 수 있어, 감사 로그·실행 기록·근거(evidence)와 연결된 작업 이력이 끊긴다. 사이클·플랜의 진행 집계가 사후에 조작되어, "무슨 일이 있었는지"를 신뢰할 수 없게 된다. 활성 플랜에서의 삭제 금지는 이 작업 추적의 무결성을 지키는 장치다.

## 코드에서 어떻게 강제되나
CLI의 태스크 Delete 명령은 "플랜이 아직 draft일 때만 허용된다"고 명시되어 있다(`cli/src/main.rs`의 Delete 변형 설명). 삭제 가부 판정은 데몬이 해당 태스크의 플랜 상태를 확인해 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(삭제 정책의 근거 결정) 연결 필요
- [ ] OPEN: 데몬 측 삭제 거부 로직 구현 위치 확인 필요
