---
id: endpoint.plan
kind: Endpoint
title: clawket plan (플랜 라이프사이클·가져오기/내보내기)
definition: "clawket plan — 플랜을 만들고 보고 수정·삭제하며, approve 로 active 전환하고 complete 로 종료하고, 마크다운에서 가져오거나 md/json/yaml 로 내보내는 CLI 명령 그룹."
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.plan, type: mutates, note: "플랜 엔티티를 생성·수정·삭제하고 draft→active→completed 상태를 전이한다" }
  - { to: concept.execution-envelope, type: reads, note: "strict import/export 는 19-field 실행 봉투와 의존 그래프를 라운드트립한다" }
  - { to: concept.task, type: reads, note: "플랜의 모든 태스크가 종료되면 플랜이 자동 완료된다" }
governedBy: [invariant.one-active-plan, invariant.start-needs-active-plan-cycle, invariant.db-is-sot-plan-markdown-view, invariant.terminal-auto-cascade]
impacts: [domain.work-management]
consumedBy: [endpoint.hook-plan-sync, component.claude-adapter]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

`clawket plan` 은 **승인된 의도를 담는 컨테이너(로드맵)** 인 플랜을 다루는 명령 그룹이다. 플랜은 `draft → active → completed` 상태를 가지며, **active 가 되어야 그 아래 태스크를 시작(in_progress)** 할 수 있다. 사람·에이전트가 작업 묶음의 방향을 확정하고 그 묶음을 실행 가능 상태로 올릴 때 쓴다.

제공하는 동작:
- **만들기(create)**: 제목과 소속 프로젝트로 플랜을 만든다. 플랜은 `draft` 로 시작한다. draft 상태에서도 태스크를 만들어 둘 수는 있으나(todo), 시작은 못 한다.
- **보기/목록(view/list)**: 단일 플랜 상세, 또는 프로젝트·상태로 거른 목록을 본다.
- **수정(update)**: 제목·설명·상태를 바꾼다. (draft→active 전이는 전용 `approve` 를 쓰는 것이 정석.)
- **삭제(delete)**: 플랜을 지운다.
- **승인(approve)**: draft 플랜을 active 로 올린다 — 태스크 시작의 전제 조건.
- **완료(complete)**: active 플랜을 completed 로 닫는다. 플랜은 그 안의 모든 태스크가 종료(done/cancelled)되면 자동 완료되므로, 이 명령은 주로 조기 종료용이다.
- **가져오기(import)**: 마크다운 파일에서 플랜을 만든다. strict 모드는 19-field 실행 봉투와 의존 그래프까지 한 줄씩 검증해 영속한다(loose 모드는 봉투·의존 그래프 무시).
- **내보내기(export)**: 플랜을 md/json/yaml 로 출력한다. DB가 단일 진실이고 마크다운은 생성된 뷰다 — 봉투 편집 후 스냅샷을 다시 만들거나 다른 도구로 넘길 때 쓴다.

## 요청 / 응답

- **만들 때** 보내는 정보: 제목(필수), 소속 프로젝트(필수), 설명, 소스 라벨(manual/import), 소스 파일 경로.
- **목록**은 프로젝트·상태 필터로 추린 플랜들을 반환하며, 각 항목은 제목·상태·소속 프로젝트를 담는다.
- **import** 는 플랜 마크다운 파일 경로와 붙일 프로젝트(없으면 cwd로 추론·생성)를 받고, dry-run 이면 만들지 않고 파싱 결과만 미리 보여준다. strict 면 봉투/의존 검증 결과까지 반영한다.
- **export** 는 플랜 ID와 출력 형식을 받아 그 형식의 직렬화 결과를 stdout 또는 파일로 준다. 옵션으로 첨부 지식을 부록으로 포함할 수 있다(기본 미포함, 크기 예측 가능성 유지).

## 권한 / 제약

- 프로젝트당 active 플랜은 동시에 하나뿐 — `invariant.one-active-plan`.
- 태스크 시작에는 active 플랜이 필수 — `invariant.start-needs-active-plan-cycle`.
- DB가 단일 진실, 마크다운은 손으로 고치는 문서가 아니라 생성 뷰 — export 의 round-trip parity 는 strict 파서로만 보장된다(`invariant.db-is-sot-plan-markdown-view`).
- 마지막 비종료 태스크가 종료되면 플랜이 자동으로 completed 로 cascade 된다 — `invariant.terminal-auto-cascade`.

## provenance

- 명령 정의·핸들러: `cli/src/main.rs`(Commands::Plan, PlanAction). 영속·상태 전이·import/export 변환은 데몬 API로 위임.
- 제공 컴포넌트: `component.cli` + `component.daemon`.
- 재구현 시 보존해야 할 의미: draft/active/completed 의 게이트(active 가 태스크 시작 자격), strict import 의 봉투·의존 검증, export 가 DB→마크다운 생성 뷰라는 단방향성.

## 미확정 (OPEN)

- [ ] OPEN: loose vs strict import 의 정확한 파싱 차이(어떤 줄을 무시/검증하는지)는 import 파서 정독으로 확인 필요.
- [ ] OPEN: update 로 직접 status 를 바꾸는 경로와 approve/complete 의 차이(검증 누락 여부)는 데몬 라우트 확인 필요.
