---
id: endpoint.plans-import
kind: Endpoint
title: 플랜 가져오기 (POST /plans/import)
definition: 디스크의 플랜 파일을 읽어 한 프로젝트의 플랜·유닛·태스크 구조로 등록하는 데몬 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/import_export.rs
relatesTo:
  - { to: concept.plan, type: mutates, note: "파일 내용을 플랜으로 등록한다" }
  - { to: concept.unit, type: mutates, note: "플랜에 속한 유닛도 함께 등록한다" }
  - { to: concept.task, type: mutates, note: "유닛 아래 태스크까지 한 번에 만든다" }
  - { to: endpoint.cli-plan, type: backed-by, note: "plan import 서브커맨드의 백엔드 경로" }
governedBy: [invariant.db-is-sot-plan-markdown-view]
impacts: [concept.plan, concept.unit, concept.task]
consumedBy: [endpoint.cli-plan]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

이미 작성해 둔 플랜 파일(디스크 상의 파일 경로)을 통째로 읽어, 그 안에 적힌 로드맵을 Clawket의 작업 구조 — 플랜 한 건과 그에 딸린 유닛들, 그리고 유닛 아래 태스크들 — 로 한 번에 등록하는 입구다. 사람이나 에이전트가 외부에서 다듬은 계획서를 작업 관리 안으로 들여올 때, 또는 다른 워크스페이스에서 내보낸 플랜을 옮겨올 때 쓴다. 파일 경로를 다루는 일반 가져오기 경로이며, Plan Mode 출력을 인라인으로 검증하는 엄격 경로(별도 엔드포인트)와는 역할이 갈린다.

## 요청 / 응답

보내는 정보는 "어느 파일을 / 어느 프로젝트에 / 정말로 쓸지 아니면 미리보기만 할지"다. 구체적으로 가져올 플랜 파일의 위치, 대상 프로젝트의 지정(프로젝트 이름이나 현재 작업 디렉터리 둘 중 하나로 어느 프로젝트인지 식별), 이 가져오기가 어디서 비롯됐는지를 알려주는 출처 표시, 그리고 실제 등록 없이 결과만 시험해 보는 미리보기(dry-run) 여부를 함께 보낸다. 출처 표시는 나중에 감사 기록에서 "이 플랜이 어떤 경로로 들어왔는지"를 구분하는 데 쓰이며, 따로 지정하지 않으면 일반 가져오기로 표시된다.

돌려주는 정보는 가져오기 결과 요약이다. 무엇이 새로 만들어졌고 무엇이 건너뛰어졌는지 등 플랜·유닛·태스크 등록 결과를 담는다. 미리보기로 호출하면 실제 데이터는 만들지 않고 "이렇게 들어올 것"이라는 예상 결과만 같은 형태로 돌려준다.

## 권한 / 제약

로컬 데몬에 붙은 호출자(CLI·에이전트)가 사용하며, 별도 외부 인증 없이 로컬 경계 안에서 동작한다. 가져온 결과는 데이터베이스에 플랜·유닛·태스크로 영구 기록되고, 사람이 읽는 마크다운 등은 그 데이터베이스 상태를 비추는 뷰로 다뤄진다(데이터베이스가 단일 진실원천). 미리보기 모드에서는 아무 것도 영구 기록되지 않으므로, 실제로 들여오기 전에 결과를 안전하게 확인할 수 있다.

## provenance

데몬의 가져오기/내보내기 라우터(`daemon/src/routes/import_export.rs`)가 `/plans/import` 경로로 이 동작을 제공하며, 파일을 읽어 플랜을 등록하는 실제 처리는 데몬 내부의 플랜 가져오기 로직에 위임한다. 제공 주체는 clawketd 데몬 컴포넌트다.

## 미확정 (OPEN)
- [ ] OPEN: 가져오기 응답이 보고하는 항목(생성·건너뜀·갱신의 정확한 구분과 의미)을 사람 언어로 더 확정할 것.
- [ ] OPEN: 출처 표시 값의 표준 어휘(일반 가져오기 vs 다른 경로)와 그 각각이 감사 기록에서 갖는 의미를 owner와 확인.
