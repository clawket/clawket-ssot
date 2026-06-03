---
id: endpoint.plans-get
kind: Endpoint
title: 데몬 GET /plans/{id}
definition: "단일 플랜의 현재 형상(제목·설명·상태·출처·소속 프로젝트 등)을 ID로 조회하는 읽기 엔드포인트."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/plans.rs]
relatesTo:
  - { to: concept.plan, type: reads, note: "플랜 한 건을 그대로 읽어 반환" }
governedBy: []
impacts: []
consumedBy: [component.web-dashboard, endpoint.cli-plan]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

플랜 식별자 하나를 받아 그 플랜의 현재 상태를 그대로 돌려주는 단순 조회다. 웹 대시보드가 플랜 상세를 그릴 때, CLI가 `plan view`로 한 건을 확인할 때, 또는 다른 작업 흐름이 어떤 플랜의 상태(초안인지 활성인지 완료인지)를 알아야 할 때 호출된다. 작업 워크플로우에서 "이 플랜이 지금 어떤 단계인가"를 묻는 가장 기본적인 질문에 답한다.

## 요청 / 응답

요청에는 조회 대상 플랜의 식별자만 담는다. 응답으로는 그 플랜 한 건의 전체 형상 — 제목, 설명, 현재 상태(초안/활성/완료), 플랜이 어떻게 만들어졌는지를 가리키는 출처와 출처 경로, 소속 프로젝트, 승인 시각 등 — 을 돌려준다. 식별자에 해당하는 플랜이 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약

로컬 데몬에 붙은 클라이언트(웹 대시보드·CLI)만 호출할 수 있는 로컬 전용 엔드포인트다. 별도의 사용자 권한 구분은 없고, 존재하지 않는 식별자에 대해서는 404 성격의 "찾을 수 없음"으로 답하는 것이 유일한 예외 처리다. 상태를 바꾸지 않는 순수 읽기이므로 부수효과가 없다.

## provenance

라우터 정의와 핸들러는 `daemon/src/routes/plans.rs`의 `/plans/{id}` GET 경로(`get_one`)에 있으며, 데이터 접근은 플랜 저장소 계층(`plans::get`)을 거친다. 제공 주체는 로컬 clawketd 데몬(component.daemon)이다.

## 미확정 (OPEN)
- (없음)
