---
id: endpoint.units-get
kind: Endpoint
title: 단일 유닛 조회 (GET /units/{id})
definition: 식별자로 지정한 유닛 한 건의 상세 정보를 돌려주는 데몬 조회 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/units.rs
relatesTo:
  - { to: concept.unit, type: reads, note: "유닛 한 건의 상세를 조회한다" }
  - { to: endpoint.cli-unit, type: backed-by, note: "unit view 서브커맨드의 백엔드 경로" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-unit, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

특정 유닛 하나를 식별자로 콕 집어 그 상세 내용을 가져오는 조회 입구다. 목록에서 고른 유닛의 세부를 펼쳐 보거나, 유닛을 다루기 전 현재 상태를 확인할 때 쓴다. 읽기 전용이다.

## 요청 / 응답

보내는 정보는 보고 싶은 유닛의 식별자 하나다. 돌려주는 정보는 그 유닛의 상세 — 무엇인지(제목·목표), 어느 플랜에 속하는지, 표시 순서나 실행 방식 같은 그룹핑 속성 — 다. 해당 식별자의 유닛이 없으면 "찾을 수 없음"으로 응답한다. 유닛은 상태·승인 단계가 없는 순수 그룹핑 단위이므로 진행 상태 값은 포함되지 않는다.

## 권한 / 제약

로컬 데몬 경계 안에서 조회용으로 동작하며 별도 외부 인증은 없다. 읽기 전용이라 데이터를 변경하지 않는다. 존재하지 않는 식별자는 정상적인 "찾을 수 없음" 응답으로 처리된다.

## provenance

데몬의 유닛 라우터(`daemon/src/routes/units.rs`)가 `/units/{id}` 경로의 조회 동작으로 제공하며, 단건 조회는 데몬 내부의 유닛 저장소 조회 로직에 위임한다. 제공 주체는 clawketd 데몬이고, 호출 주체는 CLI의 unit 명령과 웹 대시보드다.

## 미확정 (OPEN)
- [ ] OPEN: 단건 응답이 목록 항목과 다른 추가 정보(연관 태스크 요약 등)를 포함하는지 owner와 확인.
