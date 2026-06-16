---
id: endpoint.cli-migrate
kind: Endpoint
title: clawket migrate
definition: "보류 중인 데이터 스키마 변경을 적용하는 CLI 명령. 평소에는 데몬이 시작할 때 자동으로 처리하므로 수동 호출은 점검·단계 적용이 필요한 경우에 한한다."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: component.sqlite-storage
    type: mutates
    note: "스키마 마이그레이션을 적용하면 로컬 SQLite 저장소의 구조가 갱신된다."
  - to: component.daemon
    type: backed-by
    note: "마이그레이션 실행 책임은 데몬이 진다 — CLI는 데몬에 적용을 요청한다."
governedBy: []
impacts:
  - component.sqlite-storage
consumedBy:
  - persona.human-operator
  - persona.llm-coding-agent
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
사용자가 보관 중인 모든 작업 데이터(프로젝트·플랜·태스크·지식 등)는 로컬 저장소에 일정한 구조로 담겨 있다. 제품이 새 버전으로 올라가며 그 구조가 바뀌면, 기존 데이터를 새 구조에 맞게 옮겨 주는 절차가 필요하다 — 이것이 마이그레이션이다.

`clawket migrate`는 아직 적용되지 않은 그런 구조 변경을 적용한다. 다만 보통의 사용자는 이 명령을 직접 칠 일이 거의 없다. 데몬이 켜질 때 보류 중인 변경을 스스로 적용하기 때문이다. 이 명령은 그 자동 적용 흐름과 별개로, 변경 내용을 미리 들여다보거나 단계적으로 적용하고 싶은 운영자·점검 상황을 위한 수동 진입점이다.

## 요청 / 응답
보내는 정보는 "미리보기만 할지" 여부 하나다. 미리보기 모드를 켜면 실제로 적용하지 않고 어떤 변경이 보류 중인지만 확인한다. 끄면 보류 중인 변경을 실제로 적용한다.

돌려주는 정보는 마이그레이션 처리 결과다 — 무엇이 적용되었는지(또는 미리보기 시 무엇이 적용될 예정인지)를 사람이 확인할 수 있는 형태로 보고한다.

## 권한 / 제약
- 로컬 데몬이 살아 있어야 동작한다(실제 적용 주체가 데몬이므로).
- 데이터 구조 자체를 바꾸는 작업이므로, 적용 전에 미리보기로 내용을 확인하는 것이 안전한 사용법이다.
- 평상시에는 데몬 시작 시 자동 적용되므로 이 명령을 수동으로 부를 필요가 없다 — 자동 흐름의 백업·점검 수단이다.

## provenance
CLI 명령 정의·핸들러는 `cli/src/main.rs`에 있다. CLI는 적용 요청을 데몬(`component.daemon`)에 위임하고, 데몬이 로컬 저장소(`component.sqlite-storage`)의 구조를 갱신한다. 마이그레이션 적용 자체는 데몬의 저장소 초기화 경로에서 일어난다.

## 미확정 (OPEN)
- [ ] OPEN: CLI가 호출하는 마이그레이션 적용 경로가 현재 데몬 라우트 구성과 정확히 어떻게 매핑되는지 확인 필요(검사 시점의 빌드 바이너리와 데몬 소스 간 명령 표면 차이 가능성). 미리보기 응답이 보류 목록을 어떤 의미 단위로 보여 주는지도 확인 필요.
