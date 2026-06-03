---
id: endpoint.units-list
kind: Endpoint
title: 유닛 목록 조회 (GET /units)
definition: 한 플랜에 속한 유닛들의 목록을 돌려주는 데몬 조회 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/units.rs
relatesTo:
  - { to: concept.unit, type: lists, note: "유닛 컬렉션을 조회한다" }
  - { to: concept.plan, type: reads, note: "플랜 기준으로 그 안의 유닛을 추린다" }
  - { to: endpoint.cli-unit, type: backed-by, note: "unit list 서브커맨드의 백엔드 경로" }
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

한 플랜 안에 묶여 있는 유닛(태스크를 그룹핑하는 단위)들을 한꺼번에 훑어보는 조회 입구다. 사람이 대시보드에서 플랜의 구성을 펼쳐 보거나, 에이전트가 어떤 유닛들에 태스크를 배치할지 파악할 때 쓴다. 읽기 전용이라 상태를 바꾸지 않는다.

## 요청 / 응답

보내는 정보는 어느 플랜의 유닛을 볼지 가리키는 플랜 지정 한 가지다(선택). 플랜을 지정하면 그 플랜에 속한 유닛으로 범위를 좁히고, 지정하지 않으면 범위 제한 없이 유닛 목록을 돌려준다.

돌려주는 정보는 유닛 항목들의 목록이다. 각 항목은 그 유닛이 무엇인지(제목·목표)와 어느 플랜에 속하는지, 그리고 표시 순서나 실행 방식 같은 그룹핑 속성을 담는다. 유닛은 그 자체로 상태나 승인 단계를 갖지 않는 순수 그룹핑 단위이므로(상태 없음이 불변식), 목록에도 진행 상태 같은 라이프사이클 값은 들어 있지 않다.

## 권한 / 제약

로컬 데몬 경계 안에서 조회용으로 동작하며 별도 외부 인증은 없다. 읽기 전용이라 어떤 데이터도 변경하지 않는다.

## provenance

데몬의 유닛 라우터(`daemon/src/routes/units.rs`)가 `/units` 경로의 조회 동작으로 제공하며, 플랜 기준 필터링은 데몬 내부의 유닛 저장소 조회 로직에 위임한다. 제공 주체는 clawketd 데몬이고, 호출 주체는 CLI의 unit 명령과 웹 대시보드다.

## 미확정 (OPEN)
- [ ] OPEN: 목록 항목이 노출하는 속성(실행 방식·순서 등)의 사람 언어 의미를 더 확정할 것.
- [ ] OPEN: 플랜 지정 없이 호출했을 때의 기본 범위(전체 vs 활성 플랜 한정)를 owner와 확인.
