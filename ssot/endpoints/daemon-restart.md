---
id: endpoint.daemon-restart
kind: Endpoint
title: 데몬 재시작 명령
definition: 로컬 데몬을 중지한 뒤 다시 띄우는 명령. 중지 동작과 기동 동작을 순서대로 묶어 실행한다.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/daemon.rs
relatesTo:
  - to: endpoint.daemon-stop
    type: calls
    note: 먼저 중지 동작을 수행한다(정상→강제 단계 상향, pid·포트 정리 포함).
  - to: endpoint.daemon-start
    type: calls
    note: 중지 후 기동 동작으로 데몬을 다시 띄운다.
  - to: component.daemon
    type: calls
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

이 명령은 로컬 데몬을 한 번에 재시작한다. 새 동작을 따로 갖지 않고 중지 동작을 먼저 수행한 뒤 기동 동작을 이어서 실행하는, 두 명령의 합성이다. 사람이 데몬을 새 상태로 갈아끼우고 싶을 때 쓴다.

중지 단계가 정상 종료에 실패할 때 강제 종료로 단계를 올리고 남은 pid·포트 파일을 정리하도록 되어 있기 때문에, 옛 데몬이 깨끗이 정리된 뒤에 새 데몬이 떠야 한다. 이 정리가 없으면 재시작이 옛 pid 를 보고 "이미 실행 중"이라며 새로 띄우지 않는 사일런트 실패가 생긴다 — 중지 단계가 이를 방지하도록 설계되어 있다.

## 요청 / 응답

입력 인자는 없다. 응답은 중지 단계와 기동 단계가 각각 내보내는 상태 문구의 연속이다(예: 중지/강제 종료 결과에 이어 기동/준비됨 결과).

## 권한 / 제약

로컬 데몬을 멈췄다 다시 띄우는 명령이라 외부 동작이 없다. 중지 단계의 강제 종료 단계 상향은 유닉스 계열에서만 동작한다(중지 명령의 제약을 그대로 따른다). 중지가 끝내 실패하면(강제 종료 후에도 살아 있으면) 재시작도 그 지점에서 실패로 끝난다.

## provenance

중지 후 기동을 순서대로 묶는 합성 로직 근거는 cli/src/daemon.rs 의 restart 처리부.

## 미확정 (OPEN)
- [ ] 없음(현 범위에서 동작·제약 확정).
