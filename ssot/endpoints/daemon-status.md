---
id: endpoint.daemon-status
kind: Endpoint
title: 데몬 상태 조회 명령
definition: 로컬 데몬이 떠 있는지와 그 상태(PID·가동 시간·버전 등)를 보여주는 명령. 데몬 자체의 상태 보고를 받아 그대로 출력한다.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/daemon.rs
relatesTo:
  - to: component.daemon
    type: reads
    note: 데몬 자신의 status 동작을 호출해 그 출력을 그대로 전달한다.
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

이 명령은 로컬 데몬의 현재 상태를 보여준다. 직접 상태를 계산하지 않고 데몬 자신의 상태 보고(PID·가동 시간·버전 등)를 받아 그대로 출력하는 얇은 표면이다. 사람이 "데몬이 잘 떠 있나"를 확인하거나 문제 진단 시 쓴다.

## 요청 / 응답

입력 인자는 없다. 응답은 데몬이 보고한 상태 텍스트를 그대로 사람에게 보여준 것이다. 그 안에는 프로세스 식별자(PID), 가동 시간, 버전 등이 담긴다.

데몬의 상태 보고가 실패하면(예: 데몬이 떠 있지 않거나 응답하지 못하면) 이 명령은 비정상 종료 코드로 끝나, 호출하는 쪽(사람·스크립트)이 "정상 아님"을 코드로 구분할 수 있다.

## 권한 / 제약

읽기 전용 조회라 데몬 상태를 바꾸지 않는다. 로컬에서만 동작하며 외부로 나가는 것이 없다.

## provenance

데몬 status 동작 위임 근거는 cli/src/daemon.rs 의 status 처리부.

## 미확정 (OPEN)
- [ ] OPEN: 상태 보고에 정확히 어떤 항목이 포함되는지(데몬 측 status 출력 스키마)는 데몬 본체 정독으로 확정 필요.
