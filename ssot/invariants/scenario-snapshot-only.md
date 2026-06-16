---
id: invariant.scenario-snapshot-only
kind: Invariant
title: 시나리오 지식 본문은 현재 의도만 담고 변경 이력은 누출 금지다
definition: "시나리오 지식 본문은 현재 의도만 담아야 하며 changelog·변경 흔적이 본문에 섞이면 위반이고, 변경 이력은 별도 감사 지식과 취소된 QA 태스크 코멘트에만 둔다."
governs:
  - concept.scenario
  - concept.knowledge
  - capability.author-scenarios
  - domain.pdd-verification
implementedIn:
  - clawket/skills/clawket-verify-loop/SKILL.md
  - clawket/skills/clawket-scenario-refine/SKILL.md
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
relatesTo:
  - to: concept.audit-log
    type: relates-to
    note: 변경 이력은 본문이 아니라 append-only 감사 지식·태스크 코멘트로 분리 보존
  - to: concept.comment
    type: relates-to
    note: 취소된 QA 태스크 코멘트가 원본 ID·분기·사유의 영구 흔적을 보관
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

시나리오는 버전 관리되는 지식 항목으로 보관된다. 이 지식의 본문은 **지금 시점의 의도(현재 시나리오)만** 담아야 한다. "예전에는 A였는데 지금은 B다", 취소선으로 남긴 과거 흔적, changelog 같은 변경 이력을 본문에 섞는 것은 금지된다.

시나리오를 분할하거나 의도를 재정의하거나 삭제할 때, 본문에서는 그 변경의 결과만 반영한다 — 분할이면 원본 블록을 빼고 새 ID 블록을 넣고, 의도 재정의면 해당 ID 블록만 교체하고, 삭제면 그 블록을 제거한다. "왜 바꿨는지", "원본 ID가 무엇이었는지" 같은 이력은 본문이 아니라 **별도의 감사 지식(append-only)**과 **취소된 QA 태스크의 코멘트**에 둔다. 한 번 부여된 시나리오 ID는 재사용하지 않으므로, 은퇴한 ID의 추적성은 본문이 아니라 이 분리된 이력 산출물로 유지된다.

## 깨지면 무슨 일이 일어나나

본문에 복수 시점이 공존하면 "지금의 진실"이 무엇인지 모호해진다. 검증 서브에이전트가 본문을 읽고 코드와 대조할 때, 과거 의도와 현재 의도가 섞여 있으면 어느 쪽으로 판정해야 하는지 갈리고, 판정의 일관성이 무너진다. 또 같은 시나리오 지식이 라운드를 거치며 점점 변경 메모로 비대해져, 단일 시점 스냅샷이라는 시나리오의 본질이 망가진다. 이력과 현재 상태를 분리해 두면 본문은 항상 깔끔한 현재 의도로 유지되고, 추적이 필요할 때는 감사 지식을 보면 된다(단일 책임 분리).

## 코드에서 어떻게 강제되나

코드 차단이 아니라 작업 절차(스킬 문서)와 라운드 자기점검으로 강제된다. 시나리오 정제 스킬은 (1) 본문에 changelog를 넣는 것을 명시적 안티패턴으로 거부하고, (2) 변경 사유·원본 ID는 취소된 QA 태스크 코멘트와 감사 지식(라운드마다 한 행씩 append-only)에 기록하며, (3) 본문은 현재 의도만 담도록 규정한다. 검증 루프 스킬은 라운드 진입 시 "각 유닛의 시나리오 지식이 현재 의도만 담는지(이력 누출 없음)" 스폿체크를, 라운드 종료 시 자기점검 항목으로 "본문에 이력 누출 0"을 요구한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 스냅샷 전용 정책을 정한 결정 기록 노드가 SSOT에 아직 없음
- [ ] OPEN: 자동 차단(훅) 없이 절차·자기점검으로만 강제됨 — 코드 수준 가드 도입 여부 미확인
