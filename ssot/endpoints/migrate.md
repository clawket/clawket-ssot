---
id: endpoint.migrate
kind: Endpoint
title: clawket migrate
definition: "보류 중인 DB 스키마 마이그레이션을 데몬과 별도로 직접 적용하는 CLI 명령(--dry-run 으로 적용 없이 미리보기). 평소엔 데몬이 시작 시 자동 실행한다."
realizedBy:
  - component.cli
  - component.daemon-client
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: component.daemon
    type: calls
    note: "데몬에 마이그레이션 적용을 요청한다(POST /migrate)."
  - to: component.sqlite-storage
    type: mutates
    note: "보류 중인 스키마 변경을 DB 에 적용한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

`clawket migrate` 는 아직 적용되지 않은 데이터베이스 스키마 변경(마이그레이션)을 직접 적용·점검하는 명령이다. 평상시에는 데몬이 시작될 때 보류 중인 마이그레이션을 자동으로 적용하므로 사용자가 이 명령을 따로 실행할 필요는 없다. 다만 어떤 마이그레이션이 대기 중인지 미리 확인하거나, 데몬 시작과 무관하게 별도(out-of-band)로 스키마를 정렬해야 하는 상황을 위해 수동 진입점을 제공한다.

## 요청 / 응답

요청 시 미리보기 모드를 켜면 실제로 적용하지 않고 어떤 마이그레이션이 보류 중인지만 보여 준다. 미리보기를 끄면 보류 중인 마이그레이션을 실제로 적용한다. 응답은 적용 결과(또는 미리보기 모드에서는 적용 예정 목록) 정보다.

## 권한 / 제약

데몬이 실행 중이어야 한다. 미리보기 모드가 아닌 경우 데이터베이스 스키마를 실제로 변경한다. 일반 운영에서는 데몬 시작 시 자동으로 처리되므로 수동 실행은 예외적 상황(점검·진단)에 한한다.

## provenance

CLI 정의·디스패치는 `cli/src/main.rs`(Migrate 서브커맨드: dry_run 옵션을 받아 데몬의 마이그레이션 적용을 요청). 실제 스키마 적용은 데몬 측에서 수행되며 보통 데몬 시작 시 자동 실행된다. "보류 중인 DB 스키마 변경을 미리보기 또는 실제 적용으로 처리하는, 평소엔 데몬이 자동 수행하는 동작의 수동 진입점"이라는 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 마이그레이션 적용 경로가 미리보기 응답에서 보류 마이그레이션을 어떤 식별/설명으로 표현하는지를 데몬 측에서 확인할 것.
