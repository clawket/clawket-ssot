---
id: concept.plan
kind: Concept
title: Plan
definition: "승인된 의도를 담는 컨테이너(로드맵). draft → active → completed 상태를 거치며, active가 되어야 그 아래 태스크를 시작할 수 있다. 플랜은 승인된 의도의 단일 진실원천이다."
relatesTo:
  - { to: concept.project, type: belongs-to, note: "플랜은 한 프로젝트에 속한다. 프로젝트당 active 플랜은 하나만 허용." }
  - { to: concept.unit, type: contains, note: "플랜은 여러 유닛을 담는다." }
  - { to: capability.plan-and-approve, type: backed-by, note: "플랜 제안·등록·approve 행위로 실체화된다." }
  - { to: endpoint.cli-plan, type: mutates, note: "create/approve/complete/import/export 등으로 관리된다." }
  - { to: endpoint.daemon-plans-import-strict, type: mutates, note: "Plan Mode 출력을 strict 검증 후 플랜으로 등록한다." }
governedBy:
  - invariant.one-active-plan-per-project
  - invariant.plan-is-source-of-truth
  - invariant.start-needs-active-plan-cycle
  - invariant.db-is-sot-plan-markdown-view
governs: []
dependsOn: []
implementedIn:
  - daemon/src/models.rs
  - daemon/src/repo/plans.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

Plan(플랜)은 "무엇을 하기로 승인했는가"를 담는 로드맵이다. 사람이 어떤 작업 묶음을 하기로 합의·결정하면 그 의도를 플랜으로 등록하고, 플랜이 **active**가 되어야 비로소 그 안의 태스크를 시작할 수 있다. Clawket에서 플랜은 "승인된 의도의 단일 진실원천(source of truth)"이라는 위치를 가진다 — 작업의 방향성과 범위에 대한 합의는 플랜에 기록된다.

플랜은 세 상태를 가진다.

- **draft(초안)**: 막 만들어진, 아직 승인되지 않은 상태. 이 상태에서는 태스크를 시작할 수 없다.
- **active(활성)**: 승인(approve)을 거쳐 활성화된 상태. 이 상태여야 태스크 시작이 가능하다. 한 프로젝트 안에서 active 플랜은 동시에 하나만 존재할 수 있다.
- **completed(완료)**: 플랜의 모든 작업이 마무리된 상태. 완료 처리하려면 그 아래 미종료 태스크(done/cancelled가 아닌 것)가 남아 있으면 안 된다.

승인은 단순한 상태 변경이 아니라 "승인 시각(approved_at)"을 함께 기록해 의도가 언제 확정되었는지를 보존한다.

## 엔티티 (DB)

플랜 한 건은 소속 프로젝트, 제목·설명, 출처(source) 정보, 생성 시각, 승인 시각, 그리고 현재 상태(draft/active/completed)를 보존한다. 출처 정보는 이 플랜이 어디서 비롯되었는지(예: Plan Mode 출력에서 가져온 것인지)를 추적한다.

데이터베이스가 플랜의 단일 진실이며, markdown 등으로 내보낸 형태는 그 뷰(view)에 불과하다.

## API 표면

플랜은 `clawket plan` 명령군으로 생성·조회·수정·삭제하고, `approve`로 active 전환, `complete`로 완료 처리하며, markdown/json/yaml로 가져오고 내보낼 수 있다. 또한 Claude Code의 Plan Mode 종료를 가로채는 훅이 데몬의 strict 가져오기 엔드포인트를 거쳐 Plan Mode 출력을 Clawket 플랜으로 등록하도록 유도한다.

## 불변식

- **프로젝트당 active 플랜 하나**: 동시에 active일 수 있는 플랜은 프로젝트당 하나뿐(`invariant.one-active-plan-per-project`).
- **승인된 의도의 단일 진실**: 합의된 작업 의도는 플랜에 기록되며 플랜이 그 진실의 원천이다(`invariant.plan-is-source-of-truth`).
- **태스크 시작 전제**: 태스크 시작에는 active 플랜과 active 사이클이 모두 필요하다(`invariant.start-needs-active-plan-cycle`).
- **DB가 진실, markdown은 뷰**: 플랜의 단일 진실은 DB이고 내보낸 문서는 파생 뷰다(`invariant.db-is-sot-plan-markdown-view`).
- 완료 전환은 미종료 태스크가 없을 때만 허용된다.

## 구현 위치 (provenance)

플랜 엔티티의 형상은 `daemon/src/models.rs`, 상태 전이·승인·완료 가드·내보내기 로직은 `daemon/src/repo/plans.rs`에 있다. 완료 전환 시 미종료 태스크 검사와 승인 시 active+approved_at 동시 설정이 이 모듈에서 강제된다.

## 미확정 (OPEN)
- [ ] OPEN: completed 플랜의 재활성화(active로 되돌리기) 허용 여부 검증 필요.
