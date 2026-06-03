---
id: endpoint.cli-unit
kind: Endpoint
title: clawket unit (create/view/list/update/delete)
definition: 플랜 안에서 태스크를 묶는 그룹핑 엔티티인 유닛을 생성·조회·목록·수정·삭제하며, 실행 모드(sequential/parallel)로 병렬 유닛의 다중 에이전트 동시 실행을 허용하는 CLI 명령군.
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.unit, type: mutates, note: "유닛 생성·조회·목록·수정·삭제" }
  - { to: concept.plan, type: reads, note: "유닛은 특정 플랜에 소속되며 --plan 으로 지정" }
governedBy: [invariant.unit-no-status]
impacts: [domain.work-management]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket unit`은 플랜 안에서 태스크를 묶는 그룹핑 단위인 유닛을 다루는 명령군이다. 유닛은 상태도 승인 절차도 없는 순수 분류용 엔티티이며, 하위 동작은 생성·조회·목록·수정·삭제뿐이다(approve/complete 같은 라이프사이클 동작이 없다).

## 요청 / 응답
- 생성: 제목(필수)과 소속 플랜(필수), 목표 설명, 플랜 내 정렬 순서, 실행 모드를 받는다. 실행 모드는 sequential(기본)과 parallel 중 하나다.
- 조회/목록: 유닛 ID로 상세를 보거나, 플랜으로 걸러 목록을 받는다.
- 수정: 제목·목표·실행 모드를 바꾼다.
- 삭제: 유닛을 지운다.

## 권한 / 제약
- 유닛에는 상태나 승인 개념이 없다 — 순수 그룹핑 엔티티다. 따라서 태스크 시작 자격 같은 게이트는 유닛이 아니라 상위의 플랜(active)과 그 유닛에 묶인 사이클(active)이 결정한다.
- 실행 모드가 parallel인 유닛의 태스크는 여러 에이전트가 동시에 실행할 수 있다(기본 sequential은 순차 실행을 전제). 즉 모드는 병렬 작업 허용 여부를 표시하는 신호다.
- 로컬 데몬이 떠 있어야 동작한다.

## provenance
명령 정의와 하위 동작은 CLI의 `cli/src/main.rs`(UnitAction 정의와 처리)에 있다. 실제 생성·조회·수정·삭제는 모두 로컬 데몬의 유닛 HTTP API를 호출해 처리하므로 제공 주체는 CLI 바이너리와 데몬이다. 유닛에서 상태·승인이 제거된 사실은 마이그레이션(`migrations/013_drop_unit_status_approval.sql`)에서도 확인된다.

## 미확정 (OPEN)
- [ ] OPEN: parallel 모드가 실제 다중 에이전트 디스패치(검증 루프의 배치 실행 등)에서 어떻게 소비되는지의 연결은 검증 루프 노드와 교차 확인 필요.
