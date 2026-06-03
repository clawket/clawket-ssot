---
id: endpoint.audit
kind: Endpoint
title: 데몬 GET /audit — 감사 로그 조회(쓰기 금지)
definition: 변조 탐지 가능한 감사 로그를 읽기 전용으로 조회하는 엔드포인트로, 외부에서의 추가·수정·삭제는 모두 거부된다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/audit.rs]
relatesTo:
  - { to: concept.audit-log, type: reads, note: "감사 로그 테이블을 필터 조건으로 조회만 함" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

엔티티 변경 이력을 변조 탐지가 가능한 형태로 남기는 "감사 로그"를 들여다보기 위한 조회 엔드포인트다. 누가(actor) 무엇을(엔티티) 어떻게(작업 종류) 바꿨는지의 기록을 조건에 맞춰 가져온다. 주 사용자는 변경 이력을 추적·확인하려는 운영자나 진단 도구다.

핵심은 이 로그가 "읽기 전용"이라는 점이다 — 감사 기록의 신뢰성을 지키기 위해 API를 통한 추가·수정·삭제를 원천적으로 막는다.

## 요청 / 응답

조회 시 다음과 같은 필터를 선택적으로 줄 수 있다: 어떤 종류의 엔티티인지, 특정 엔티티 하나로 좁힐지, 누가 한 변경인지(actor), 어떤 작업 종류(op_type)인지, 그리고 최대 몇 건까지 받을지.

응답은 조건에 맞는 감사 기록 목록이다. 각 항목은 대상 엔티티·작업 종류·바뀐 필드와 이전/이후 값·수행 주체·발생 시점 같은 변경 사실을 담는다.

추가·수정·삭제 계열 요청(POST/PUT/DELETE/PATCH)을 보내면 "감사 로그는 불변"이라는 거부 응답(메서드 허용 안 됨)을 돌려준다.

## 권한 / 제약

- 조회만 허용된다. 쓰기 계열 메서드는 전부 거부된다(불변 로그).
- 감사 기록은 데몬 내부 변경 로직 안에서만 작성된다 — 외부 호출자는 기록을 주입하거나 바꿀 수 없다. 이것이 "변조 탐지 가능"이라는 성격을 뒷받침한다.
- 로컬 데몬에 직접 붙는 호출이므로 로컬에서만 접근 가능하다.

## provenance

감사 라우터(`daemon/src/routes/audit.rs`)가 `/audit` 경로의 조회 핸들러를 제공하고, 같은 경로의 쓰기 메서드들은 의도적으로 "메서드 허용 안 됨" 응답으로 묶여 있다. 기록 자체는 로컬 SQLite의 감사 로그 테이블(`component.daemon`이 소유, `concept.audit-log`로 모델링)에서 읽는다. 코드가 사라져도 "변경 이력을 읽기 전용으로만 조회 / 쓰기 차단으로 불변성 보장"이라는 정책만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 감사 로그의 변조 탐지 메커니즘(해시 체인 등)의 구체 동작은 감사 로그 개념 노드(`concept.audit-log`)와 저장소 구현을 함께 봐야 확정된다. 본 노드는 조회 엔드포인트의 의미·제약 범위만 다룬다.
