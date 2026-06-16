---
id: endpoint.cli-stop
kind: Endpoint
title: 데몬 중지 (clawketd stop)
definition: 실행 중인 로컬 데몬에 정상 종료(graceful shutdown) 신호를 보내 처리 중인 일을 마무리하고 깨끗이 멈추게 하는 명령이다.
realizedBy:
  - component.daemon
  - component.cli
implementedIn:
  - daemon/src/main.rs
  - cli/src/daemon.rs
relatesTo:
  - to: endpoint.cli-daemon
    type: belongs-to
    note: clawket daemon 명령군의 stop 하위 동작이다.
  - to: endpoint.daemon-health
    type: relates-to
    note: 중지 여부 판단의 기준이 되는 생존/헬스 신호와 짝을 이룬다.
governedBy: []
impacts:
  - platform.local-first
consumedBy:
  - component.cli
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
사용자나 운영 자동화가 "지금 켜져 있는 데몬을 멈춰라"라고 지시할 때 쓰는 명령이다. 데몬은 작업 관리 데이터(SQLite)를 소유하고 API·실시간 이벤트를 제공하는 백그라운드 프로세스이므로, 갑자기 죽이는 대신 진행 중인 요청을 마무리할 기회를 주는 "정상 종료"를 기본으로 한다. 업데이트·재시작·정리 작업 전에 호출된다.

## 요청 / 응답
- 보내는 것: 추가 입력 없이 "멈춰라"라는 단일 지시. 어떤 데몬을 멈출지는 로컬에 기록된 현재 실행 정보(프로세스 식별 정보·접속 포트 기록 파일)로 스스로 찾는다.
- 돌려주는 것: 사람이 읽는 한 줄 결과 메시지.
  - 애초에 실행 중이 아니면 "실행 중 아님"으로 알리고 정상 종료로 처리한다. 과거 흔적만 남은 낡은 기록이 있으면 그 흔적을 청소하고 역시 "실행 중 아님"으로 본다.
  - 살아 있으면 정상 종료를 시도하고 멈추면 중지 완료를 알린다.

## 권한 / 제약
- 로컬 전용 명령이다. 외부에서 호출하는 원격 API가 아니라, 같은 머신에서 데몬을 관리하는 운영 명령이다.
- 정상 종료는 신호를 보낸 뒤 일정 시간(약 10초) 동안 데몬이 스스로 빠질 때까지 기다린다. 처리 중이던 요청을 마무리할 유예를 준 뒤에야 종료가 확정된다.
- 데몬이 그 유예 안에 빠지지 않고 멈춰 버린(wedge) 경우, 명령은 실패로 끝나는 데 그치지 않고 강제 종료로 단계적 격상(escalation)을 한 뒤 남은 낡은 실행 기록 파일을 청소한다. 이렇게 해야 곧바로 새 데몬을 깨끗하게 다시 띄울 수 있다 — 조용히 실패한 채 낡은 프로세스를 살려 두면 재시작이 "이미 실행 중"으로 오인되는 함정이 생기기 때문이다.
- 좀비(죽었지만 식별 정보만 남은) 프로세스를 살아 있다고 오판하지 않도록, 단순한 식별 정보 존재 여부가 아니라 실제 응답 가능 여부로 생존을 판단한다.

## provenance
정상 종료의 본체(신호 발송 → 종료 폴링 → 실행 기록 파일 청소)는 데몬 자체(`daemon/src/main.rs`의 stop 처리: `stop_daemon`). 사용자 진입점과 실패 시 강제 종료 격상·낡은 파일 청소 오케스트레이션은 CLI 쪽(`cli/src/daemon.rs`의 `cmd_stop`). 어떤 데몬을 멈출지는 로컬 실행 기록(프로세스 식별·포트 기록) 파일에서 해석한다.

## 미확정 (OPEN)
- [ ] OPEN: 정상 종료 유예 시간(약 10초)·강제 종료 후 대기(약 2초) 등 구체 수치가 정책으로 고정된 값인지, 환경에 따라 조정 가능한지 owner 확인 필요.
