---
id: endpoint.unit
kind: Endpoint
title: clawket unit (플랜 내 그룹핑 유닛 관리)
definition: "clawket unit — 플랜 안에서 태스크를 묶는 순수 그룹핑 엔티티를 만들고 보고 수정·삭제하며, sequential/parallel 실행 모드를 지정하는 CLI 명령 그룹."
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.unit, type: mutates, note: "유닛 엔티티를 생성·수정·삭제하고 실행 모드를 설정한다" }
  - { to: concept.plan, type: reads, note: "유닛은 반드시 한 플랜에 속한다" }
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

`clawket unit` 은 플랜 안에서 태스크를 묶는 **순수 그룹핑 단위** 인 유닛을 다루는 명령 그룹이다. 유닛은 상태도 승인도 없고(approve 없음), 단지 태스크를 의미 단위로 묶고 실행 방식을 표시한다. 작업을 "이 묶음은 순차로, 저 묶음은 병렬로" 식으로 구조화할 때 쓴다.

제공하는 동작:
- **만들기(create)**: 제목과 소속 플랜으로 유닛을 만든다. 목표 설명, 플랜 내 정렬 순서, 그리고 실행 모드(sequential 기본 / parallel)를 줄 수 있다. parallel 유닛의 태스크는 여러 에이전트가 동시에 실행할 수 있다.
- **보기/목록(view/list)**: 단일 유닛 상세, 또는 플랜으로 거른 목록을 본다.
- **수정(update)**: 제목·목표·실행 모드를 바꾼다.
- **삭제(delete)**: 유닛을 지운다.

## 요청 / 응답

- **만들 때** 보내는 정보: 제목(필수), 소속 플랜(필수), 목표 설명, 정렬 순서, 실행 모드.
- **목록**은 플랜 필터로 추린 유닛들을 반환하며, 각 항목은 제목·목표·실행 모드·소속 플랜을 담는다.
- **수정**은 제목·목표·실행 모드 중 준 것만 바꾼다.

## 권한 / 제약

- 유닛은 상태가 없는 순수 grouping 엔티티다 — 승인 흐름이 없고, "활성/완료" 같은 상태 전이도 없다(`invariant.unit-no-status`). 따라서 사이클처럼 활성화하거나 플랜처럼 승인할 대상이 아니다.
- 유닛은 반드시 한 플랜에 속한다(create 시 `--plan` 필수).
- 실행 모드 parallel 은 그 유닛의 태스크들이 다중 에이전트 동시 실행 대상임을 뜻하지만, 모드 자체가 동시 실행을 강제하는 것은 아니라 표시·정책 신호로 작동한다.

## provenance

- 명령 정의·핸들러: `cli/src/main.rs`(Commands::Unit, UnitAction). 영속은 데몬 API로 위임.
- 제공 컴포넌트: `component.cli` + `component.daemon`.
- 재구현 시 보존해야 할 의미: 유닛이 상태 없는 그룹핑이라는 점, sequential/parallel 모드의 의미(병렬은 다중 에이전트 신호).

## 미확정 (OPEN)

- [ ] OPEN: parallel 모드가 사이클/태스크 시작 게이트나 스포너 동작에 실제로 미치는 효과가 있는지(표시용인지 강제용인지)는 데몬·어댑터 정독으로 확인 필요.
- [ ] OPEN: 유닛 삭제 시 소속 태스크/사이클의 처리(연쇄 삭제 vs 미할당)는 데몬 라우트 확인 필요.
