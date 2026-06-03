---
id: endpoint.cli-daemon
kind: Endpoint
title: clawket daemon (start/stop/status/restart/log)
definition: "로컬 clawketd 데몬의 생애주기를 다루는 CLI 명령 묶음으로, 데몬을 백그라운드로 띄우고·멈추고·상태를 보고·재시작하고·로그를 확인한다."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/daemon.rs
relatesTo:
  - to: component.daemon
    type: relates-to
    note: 이 명령이 통제 대상으로 삼는 데몬 프로세스
  - to: endpoint.daemon-health
    type: relates-to
    note: 데몬이 살아 응답하는지의 판정 근거로 소켓 연결 프로브를 사용한다
governedBy:
  - invariant.daemon-liveness-socket-probe
  - invariant.daemon-flock-single-instance
impacts: []
consumedBy:
  - component.install-gate
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

Clawket의 모든 상태를 소유하고 API를 제공하는 로컬 데몬(clawketd)을 사람이 직접 다루기 위한 명령이다. 데몬을 백그라운드로 기동(start)하거나, 멈추거나(stop), 지금 살아 있는지·언제부터 떠 있었는지·어떤 버전인지를 확인하거나(status), 한 번에 끄고 다시 켜거나(restart), 데몬이 남긴 로그를 보는(log) 다섯 가지 동작을 묶는다. 보통은 세션 시작 시 자동으로 데몬이 보장되므로 사람이 이 명령을 직접 칠 일은 많지 않지만, 데몬이 응답하지 않을 때 상태를 확인하고 되살리는 일선 도구다.

## 요청 / 응답

기동은 데몬을 백그라운드 프로세스로 분리해 띄운다 — 이때 데몬은 호출한 CLI가 곧 종료되더라도 살아남도록 터미널에서 떼어져 독립 프로세스로 재부모화된다. 상태 조회는 데몬의 프로세스 식별자·가동 시간·버전을 돌려준다. 로그 보기는 데몬이 남긴 로그에서 최근 줄(기본 50줄)을 보여 주며, 실시간 따라보기(follow) 옵션으로 새 줄이 쌓이는 대로 이어서 볼 수 있다.

데몬이 "살아 있다"는 판정은 단순히 프로세스 식별자가 존재하는지가 아니라, 데몬이 듣고 있는 유닉스 소켓에 실제로 연결이 되는지로 내린다. 강제 종료로 남은 좀비 프로세스는 식별자 존재 검사를 통과해 버려 죽은 데몬을 산 것으로 오판하게 만들기 때문에, 연결 프로브만이 신뢰할 수 있는 단일 진실이다.

## 권한 / 제약

데몬 바이너리는 환경변수 명시 경로 → 표준 후보 경로 → PATH 순으로 찾는다. 한 머신에서 데몬은 단일 인스턴스만 떠야 하며(잠금으로 보장), 멈춤 경로에서는 대상 프로세스가 아직 살아 있을 때만 종료 신호를 보낸다. 기동·정지의 성공 여부는 식별자 존재가 아니라 소켓 연결 프로브로 확정한다.

## provenance

`cli/src/daemon.rs`에서 확인했다. 데몬 바이너리 탐색 순서, 더블-포크로 데몬을 init 아래로 재부모화하는 백그라운드 기동, 소켓 연결 프로브 기반 생존 판정(`daemon_alive`)과 식별자 존재 검사(`is_running`)의 역할 분리를 코드 주석과 함께 추적했다. 명령 자체는 CLI(`component.cli`)가 노출하지만 통제 대상은 데몬 프로세스(`component.daemon`)다.

## 미확정 (OPEN)
- [ ] OPEN: restart 시 정지 완료를 기다리는 타임아웃·재시도 정책과, 기동 실패 시 사용자에게 돌려주는 진단 메시지의 정확한 형태는 추가 정독 필요.
