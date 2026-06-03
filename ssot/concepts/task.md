---
id: concept.task
kind: Concept
title: Task
definition: "직접 다루는 유일한 원자적 작업 단위. todo → in_progress → done | cancelled로 진행하며(외부 의존 시 blocked), done 전환에는 evidence(근거)가 필요하다. 시작에는 active 플랜과 active 사이클이 모두 필요하다."
relatesTo:
  - { to: concept.unit, type: belongs-to, note: "태스크는 유닛에 속하며, 유닛→플랜→프로젝트로 소속이 간접 결정된다." }
  - { to: concept.cycle, type: belongs-to, note: "태스크는 active 사이클에 배정되어야 시작할 수 있다." }
  - { to: concept.execution-envelope, type: relates-to, note: "리프 태스크에 실행 봉투가 서명되어 decompose→contract→execute 루프를 강제한다." }
  - { to: concept.scenario, type: relates-to, note: "검증 워크플로우에서 시나리오와 1:1로 매핑된다(scenario_id)." }
  - { to: concept.run, type: relates-to, note: "태스크 시작 시 훅이 실행 기록(run)을 자동 생성한다." }
  - { to: concept.ticket-number, type: relates-to, note: "ULID와 병행하는 사람이 읽는 식별자(CK-N)를 가진다." }
  - { to: capability.manage-task-lifecycle, type: backed-by, note: "생성·시작·완료·취소·차단/해제 행위로 실체화된다." }
  - { to: endpoint.cli-task, type: mutates, note: "create/complete/cancel/block/unblock/decompose/tree 등으로 관리된다." }
governedBy:
  - invariant.no-work-without-task
  - invariant.start-needs-active-plan-cycle
  - invariant.cycle-required-for-tasks
  - invariant.evidence-required-on-done
  - invariant.blocked-reason-required
  - invariant.preserve-history
  - invariant.tasks-not-project-scoped
  - invariant.task-delete-only-draft
governs: []
dependsOn: []
implementedIn:
  - daemon/src/models.rs
  - daemon/src/repo/tasks.rs
  - daemon/src/routes/tasks.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

Task(태스크)는 사람·LLM 에이전트가 **직접 다루는 유일한 작업 단위**다. 프로젝트·플랜·유닛·사이클은 모두 태스크를 둘러싼 구조이고, 실제 "일"은 태스크에서 일어난다. 태스크는 원자적이어야 한다 — 너무 크면 더 작은 하위 태스크로 분해(decompose)된다.

태스크의 라이프사이클은 다음과 같다.

- **todo**: 시작 대기.
- **in_progress**: 진행 중. todo에서 시작하며, 시작에는 **active 플랜과 active 사이클이 모두** 필요하다.
- **done**: 완료. 완료 전환에는 **근거(evidence)** 가 반드시 동반되어야 한다(예: 변경한 파일 위치나 추론 요약). todo에서 직접 done으로 갈 수도 있다.
- **cancelled**: 취소. 파괴적 삭제 대신 코멘트로 히스토리를 남기는 방식이 권장된다.
- **blocked**: 외부 의존으로 막힌 상태. blocked로 진입할 때는 **차단 사유(blocked_reason)** 가 필요하다.

재개방·재큐도 일부 허용된다(예: done→todo, blocked→in_progress). 태스크는 우선순위·복잡도·타입·담당자·티어(요구 모델 등급) 등 풍부한 메타데이터를 갖고, 검증 워크플로우에서는 시나리오 ID·QA 상태·결함 태스크 참조·근거·배치 ID 같은 필드를 함께 보존한다.

"활성 태스크 없이는 어떤 구현 작업도 하지 않는다"가 Clawket의 가장 기본적인 규칙이다.

## 엔티티 (DB)

태스크 한 건은 소속 유닛(필수), 배정된 사이클, 부모 태스크(분해 트리), 사람이 읽는 티켓 번호, 제목·본문, 우선순위·복잡도·예상 편집량·타입, 보고자·담당자·에이전트 ID, 생성/시작/완료 시각, 현재 상태, 의존 태스크 목록, 라벨, 실행 봉투 참조, 분해 정책, 요구 티어와 실제 실행 티어 및 에스컬레이션 사유, 그리고 검증용 필드(QA 상태·시나리오 ID·결함 태스크·시나리오 수정·근거·배치 ID)를 보존한다. 근거(evidence)는 크기 상한(약 4KiB)이 API 계층에서 강제된다.

태스크는 프로젝트에 직접 매달리지 않고 유닛을 통해 간접 소속된다.

## API 표면

태스크는 `clawket task` 명령군으로 생성·조회·수정·삭제하고, `complete`/`cancel`/`block`/`unblock`으로 상태를 전이하며, `append-body`로 본문을 누적하고, `search`·`decompose`·`tree`·`ancestors`·`descendants`·`stats`로 검색·분해·트리 조회를 한다. 상태 전이의 가드(유효 전이 검사, done 시 근거·후행조건 검사, blocked 시 사유 검사)는 데몬의 태스크 라우트와 리포지토리에서 강제된다.

## 불변식

- **태스크 없이 작업 금지**: 활성 태스크 없이는 변경 작업을 할 수 없다(`invariant.no-work-without-task`).
- **시작 전제**: 시작에는 active 플랜·active 사이클이 모두 필요하다(`invariant.start-needs-active-plan-cycle`, `invariant.cycle-required-for-tasks`).
- **done에 근거 필수**: 완료 전환에는 evidence가 동반되어야 한다(`invariant.evidence-required-on-done`).
- **blocked에 사유 필수**: blocked로 진입할 때 차단 사유가 필요하다(`invariant.blocked-reason-required`).
- **히스토리 보존**: 취소된 작업은 파괴적 삭제 대신 코멘트로 이력을 남긴다(`invariant.preserve-history`).
- **프로젝트 직속 아님**: 소속은 유닛→플랜→프로젝트로 간접 결정된다(`invariant.tasks-not-project-scoped`).
- **삭제는 초안만**: 태스크 하드 삭제는 아직 진행되지 않은(초안) 태스크에만 허용된다(`invariant.task-delete-only-draft`).

## 구현 위치 (provenance)

태스크 엔티티의 형상은 `daemon/src/models.rs`, 상태 전이 머신·근거/차단 사유 가드·생성 로직은 `daemon/src/repo/tasks.rs`, HTTP API 계층의 근거 크기 상한·후행조건 검사·사이클 필수화는 `daemon/src/routes/tasks.rs`에 있다.

## 미확정 (OPEN)
- [ ] OPEN: done 전환 시 평가되는 "후행조건(postconditions)"이 실행 봉투에서 어떻게 정의·해석되는지의 정확한 규칙 검증 필요.
