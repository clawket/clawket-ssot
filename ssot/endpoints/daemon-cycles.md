---
id: endpoint.daemon-cycles
kind: Endpoint
title: 사이클 관리 API (데몬)
definition: "데몬이 제공하는 사이클(Cycle) CRUD에 더해 활성화·완료·소속 태스크 조회를 담당한다. 사이클은 반드시 단위에 속하며, 활성화는 단위당 하나만 허용하고 시작 시각을, 완료는 종료 시각을 서버가 기록한다."
realizedBy:
  - component.daemon
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.cycle
    type: mutates
    note: "사이클 엔티티의 조회·생성·수정·삭제·활성화·완료를 수행한다."
  - to: concept.unit
    type: reads
    note: "사이클은 반드시 단위에 속한다. 단위 없이 생성하면 거부된다."
  - to: concept.task
    type: reads
    note: "특정 사이클에 속한 태스크 목록을 따로 제공한다(진행률 계산 근거)."
governedBy:
  - invariant.one-active-cycle-per-unit
  - invariant.start-needs-active-plan-cycle
  - invariant.cycle-required-for-tasks
impacts:
  - concept.cycle
  - concept.unit
  - concept.task
consumedBy:
  - component.renderer
  - component.daemon-client
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

사이클은 단위 안에서 "지금 돌고 있는 한 회차"를 나타내는 작업 흐름 층이다. 태스크를 실제로 시작하려면 그 태스크가 속한 사이클이 활성 상태여야 하므로, 활성화 동작이 이 API의 중심이다.

- 목록·단건 조회: 프로젝트 또는 단위 범위로 사이클들을 보거나 한 건을 본다.
- 생성: 프로젝트와 단위(둘 다 필수)와 제목으로 사이클을 만든다. 목표·정렬 순서는 선택. 단위 없이 만들려 하면 거부된다.
- 수정: 제목·목표·상태를 바꾼다. 상태는 계획중·활성·완료.
- 활성화: 계획중 사이클을 활성으로 올리고 시작 시각을 서버가 기록한다. 단위당 활성 사이클은 하나만 허용한다.
- 완료: 사이클을 완료로 바꾸고 종료 시각을 서버가 기록한다. 완료는 비가역 — 다시 일하려면 새 사이클을 만든다.
- 소속 태스크 조회: 한 사이클에 묶인 태스크들만 따로 가져온다(타임라인의 사이클 진행률 계산에 쓰인다).

## 요청 / 응답

- 목록: 프로젝트·단위 범위(선택) → 사이클 목록.
- 단건: 사이클 지정 → 사이클 한 건.
- 생성: 프로젝트·단위·제목(필수) + 목표·순서(선택) → 만들어진 사이클(계획중 상태로 시작). 단위 누락 시 거부.
- 수정: 바꿀 항목만(제목/목표/상태). 단, 계획중 사이클을 상태 수정만으로 "활성"으로 바꾸려 하면 거부되고 "활성화를 쓰라"는 안내가 돌아온다 — 활성화 경로를 통해야 시작 시각이 제대로 남기 때문이다. "목표"는 그대로 둠/비움/설정 세 의도를 구분해 보낸다.
- 활성화: 대상 지정(본문 없음) → 활성으로 전이되고 시작 시각이 기록된 사이클. 이미 활성인 사이클에 다시 호출해도 안전(멱등). 같은 단위의 이전 활성 사이클을 자동으로 끝내 주지는 않으므로, 필요하면 호출자가 먼저 그것을 완료해야 한다.
- 완료: 대상 지정(본문 없음) → 종료 시각이 기록된 완료 사이클.
- 소속 태스크: 사이클 지정 → 그 사이클의 태스크 목록.

## 권한 / 제약

- "단위당 활성 사이클은 하나"가 강제된다. 활성화가 이를 보장하는 지점이다.
- 태스크는 반드시 어떤 사이클에 속해야 한다 — 사이클 없이는 태스크를 만들 수 없다.
- 활성 사이클이 없으면 태스크를 시작할 수 없다(활성 계획과 함께 작업 시작의 전제).
- 계획중→활성 전이는 상태 수정이 아니라 전용 활성화 경로로만 해야 한다(시작 시각 기록 보장). 완료는 전용 완료 경로가 종료 시각을 남기는 정식 경로다.
- 완료는 비가역. 완료된 사이클은 재활성화 불가 — 새 사이클 생성이 정답.
- 이전 활성 사이클의 자동 종료는 없다 — 단위 내 활성 1개를 지키려면 호출자가 정리해야 한다.

## provenance

- 클라이언트 측 계약: `desktop/apps/desktop/src/data/api.ts` (`listCycles` / `getCycle` / `createCycle` / `updateCycle` / `activateCycle` / `completeCycle` / `deleteCycle` / `listCycleTasks`).
- 제공 컴포넌트: `component.daemon`.
- 강제 규칙: `invariant.one-active-cycle-per-unit`(단위당 활성 1개), `invariant.cycle-required-for-tasks`(태스크는 사이클 필수), `invariant.start-needs-active-plan-cycle`(작업 시작의 전제).
- 대상 엔티티: `concept.cycle`, 소속 관계는 `concept.unit` / `concept.task`.

## 미확정 (OPEN)
- [ ] OPEN: 사이클 삭제가 소속 태스크에 미치는 연쇄 효과(거부/이동/취소)를 데몬 핸들러에서 확인.
- [ ] OPEN: `listCycleTasks`와 태스크 목록(cycle_id 필터) 두 경로의 결과가 동일한지, 진행률 계산용으로 어떤 추가 정보가 붙는지 확인.
