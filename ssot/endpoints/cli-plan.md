---
id: endpoint.cli-plan
kind: Endpoint
title: clawket plan (create/view/list/update/delete/approve/complete/import/export)
definition: 플랜을 생성·조회·수정·삭제하고, approve 로 draft→active 전환해 태스크 시작 자격을 부여하며, markdown/json/yaml 로 내보내거나 markdown 파일에서 가져오는 CLI 명령군.
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.plan, type: mutates, note: "플랜 생성·조회·목록·수정·삭제·승인·완료·임포트·익스포트" }
  - { to: concept.project, type: reads, note: "플랜은 특정 프로젝트에 소속되며 --project 로 지정" }
governedBy: [invariant.start-needs-active-plan-cycle, invariant.one-active-plan-per-project, invariant.db-is-sot-plan-markdown-view]
impacts: [domain.work-management]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket plan`은 작업의 의도를 담는 컨테이너인 플랜을 다루는 명령군이다. 플랜은 처음 만들 때 초안(draft) 상태이며, 승인(approve)되어 active가 되어야 그 아래 태스크를 실제로 시작할 수 있다. 하위 동작은 생성·조회·목록·수정·삭제, 승인(draft→active), 완료(active→completed), 그리고 markdown 파일에서의 임포트와 markdown/json/yaml로의 익스포트다.

## 요청 / 응답
- 생성: 제목(필수)과 소속 프로젝트(필수), 설명, 출처(직접 작성 또는 임포트), 출처 파일 경로를 받는다. 갓 만든 플랜은 draft다. draft 아래에도 태스크를 만들 수는 있으나(todo 상태) 시작은 안 된다.
- 조회/목록: 플랜 ID로 상세를 보거나, 프로젝트·상태(draft/active/completed)로 걸러 목록을 받는다.
- 수정: 제목·설명·상태를 바꾼다(단 draft→active 전환은 전용 approve 명령을 쓴다).
- 승인: 초안 플랜을 active로 올린다 — 태스크를 시작하기 위한 필수 단계다.
- 완료: active 플랜을 completed로 닫는다. 플랜에 속한 모든 태스크가 종료 상태(done/cancelled)에 도달하면 자동으로도 완료되므로, 이 명령은 조기 종료용이다.
- 임포트: markdown 파일을 읽어 플랜으로 등록한다. 프로젝트를 함께 지정할 수 있고(없으면 작업 디렉터리로 추정), 실제 생성 없이 파싱 결과만 미리 보는 dry-run, 그리고 19필드 실행 봉투·의존 그래프까지 한 줄씩 검증해 보존하는 strict 모드를 고를 수 있다(기본은 봉투 항목을 무시하는 loose 모드).
- 익스포트: 플랜을 markdown(기본)·json·yaml로 내보낸다. 파일로 저장하거나(상위 폴더 자동 생성) 화면으로 출력하며, 플랜에 붙은 지식 항목을 부록으로 포함하는 옵션도 있다.

## 권한 / 제약
- DB가 단일 진실 원천이고 markdown은 그로부터 생성되는 뷰다 — 손으로 고치는 문서가 아니라, 봉투를 수정한 뒤 `plan export --format md`로 스냅샷을 다시 만들어 쓰는 식이다. strict 임포트는 이 markdown 익스포트와 왕복 일치(round-trip parity)를 보장하는 경로다.
- 태스크는 그 플랜이 active일 때만 시작될 수 있으므로, approve는 작업 시작 게이트 역할을 한다.
- 로컬 데몬이 떠 있어야 동작한다.

## provenance
명령 정의와 하위 동작은 CLI의 `cli/src/main.rs`(PlanAction 정의와 처리)에 있다. 실제 생성·승인·완료·임포트·익스포트는 모두 로컬 데몬의 플랜 HTTP API를 호출해 처리하므로 제공 주체는 CLI 바이너리와 데몬이다.

## 미확정 (OPEN)
- [ ] OPEN: strict/loose 파싱이 실행 봉투(execution-envelope)·의존 그래프를 어디까지 검증·보존하는지의 세부 규칙은 봉투 노드와 교차 확인 필요.
