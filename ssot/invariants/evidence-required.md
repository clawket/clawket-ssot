---
id: invariant.evidence-required
kind: Invariant
title: 상태 전이에 evidence 필수
definition: "특정 상태 전이(대표적으로 작업 완료)에는 근거 — 코드 위치(파일:줄)나 추론 요약 — 가 반드시 첨부되어야 하며, 근거가 없으면 데몬이 그 전이를 거부한다."
confidence: inferred
lastVerified: 2026-06-04
lifecycle: active
owner: TBD
governs:
  - concept.evidence-required
  - concept.task-evidence
  - concept.task-status
  - capability.manage-task-lifecycle
implementedIn:
  - daemon/src/routes/error.rs
  - daemon/migrations/022_evidence_batch.sql
decidedBy: []
crossesBoundary: true
tags:
  - status:active
  - type:invariant
relatesTo:
  - to: invariant.evidence-on-done
    type: relates-to
    note: 작업 완료 시점에 한정해 같은 근거 필수 규칙을 적용하는 자매 불변식.
  - to: invariant.error-code-stability
    type: relates-to
    note: 거부 시 외부에 알리는 코드 문자열이 안정 보장 대상이다.
---

## 제약

작업을 끝났다고 표시하는 것 같은 특정 상태 전이는 "왜 끝났다고 보는가"의 근거 없이는 허용되지 않는다. 근거는 결과를 확인할 수 있는 코드 위치(어느 파일 몇 번째 줄) 또는 그에 준하는 추론 요약 형태로 함께 제출되어야 한다. 이 규칙은 일을 끝냈다는 주장이 검증 가능한 흔적과 함께 남도록 강제해, 근거 없는 "다 됐다" 선언을 막는다.

이 제약은 데몬·명령줄·웹·LLM 훅 어느 경로로 전이를 시도하든 동일하게 적용되어 시스템 경계를 가로지른다. 근거를 빠뜨리면 데몬이 전이를 거절하면서 "근거가 필요하다"는 뜻의 전용 오류 코드를 돌려준다.

## 깨지면 무슨 일이 일어나나

근거 필수가 풀리면 작업이 아무 확인 흔적 없이 완료로 넘어갈 수 있다. 그러면 나중에 그 작업이 실제로 무엇을 어떻게 만족시켰는지 추적할 수 없고, 검증 루프에서 "끝났다"고 표시된 항목이 사실은 검증되지 않은 채 통과하게 된다. 즉 작업 기록의 신뢰도가 무너진다 — 완료라는 상태가 더 이상 "근거로 뒷받침된 완료"를 보장하지 못한다.

## 코드에서 어떻게 강제되나

근거 누락은 데몬에서 잘못된 요청으로 거부되며, 외부에는 근거 필요를 뜻하는 전용 코드(daemon/src/routes/error.rs의 코드 분기)로 전달된다. 이 코드는 명령줄·웹·LLM 훅이 직접 보고 분기하는 외부 약속의 일부라 함부로 바꾸면 안 된다(에러 코드 안정성 불변식과 짝을 이룬다). 근거 데이터를 보관할 수 있도록 저장 스키마도 갖춰져 있다(daemon/migrations/022_evidence_batch.sql).

상위 프로젝트의 규칙 문서들(CLAUDE.md, 기여 가이드, 훅 강제 문서)도 done 전환에 근거가 필수임을 명문화하고 있어, 데몬 단의 거부와 훅 단의 사전 차단이 함께 작동하는 구조다. 다만 이 인벤토리에서 확인 가능한 강제 지점은 데몬의 거부 경로와 저장 스키마이며, 그 외 경로의 강제 여부는 별도 노드에서 다룬다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 근거 필수 정책을 도입한 결정 노드가 인벤토리에 없어 미확정.
- [ ] OPEN: 데몬 거부 외 경로(훅 사전 차단)의 강제 범위는 해당 훅 노드에서 별도 확인 필요.
