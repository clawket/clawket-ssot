---
id: endpoint.cli-restart
kind: Endpoint
title: 데몬 재시작 (clawketd restart)
definition: 돌고 있는 데몬을 멈춘 뒤 새 데몬을 다시 띄우는 명령으로, 멈추기에 실패해도 차가운 재시작이 진행되도록 동작한다.
realizedBy:
  - component.daemon
  - component.cli
implementedIn:
  - daemon/src/main.rs
  - cli/src/daemon.rs
relatesTo:
  - to: endpoint.cli-daemon
    type: belongs-to
    note: clawket daemon 명령군의 restart 하위 동작이다.
  - to: endpoint.cli-stop
    type: relates-to
    note: 재시작은 중지 동작을 먼저 수행한 뒤 다시 띄운다.
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
데몬을 한 번에 "껐다 켜는" 명령이다. 새 버전 적용, 설정 변경 반영, 또는 어딘가 막혀(wedge) 응답하지 못하는 데몬을 깨끗한 상태로 되돌릴 때 쓴다. 내부적으로는 먼저 중지 동작을 수행하고 이어서 시작 동작을 수행하는 두 단계의 조합이다.

## 요청 / 응답
- 보내는 것: 추가 입력 없는 단일 재시작 지시. 새로 띄울 데몬의 기동 옵션(접속 포트·DB 경로 등)은 시작 단계가 기본값/설정으로 결정한다.
- 돌려주는 것: 중지·시작 각 단계의 사람이 읽는 결과 메시지. 새 데몬이 응답 가능 상태가 되면 "시작됨"을, 아직 준비 중이면 "시작 중"으로 알려 호출자(예: 세션 시작 훅)를 무한정 막지 않는다.

## 권한 / 제약
- 로컬 전용 운영 명령이다. 외부 원격 호출이 아니다.
- 중지 단계는 "최선 노력(best-effort)"이다. 멈추기에 실패하더라도 차가운 재시작(cold restart)이 이어서 진행되도록 설계되어, 막힌 데몬을 강제로라도 정리하고 새로 띄울 수 있게 한다. 막힌 데몬은 강제 종료로 격상하고 남은 낡은 실행 기록 파일을 청소한 뒤 새 데몬을 깨끗이 기동한다.
- 재시작 시 데몬은 새 인증 토큰을 새로 발급한다(토큰은 매 기동마다 회전). 따라서 이전 토큰에 의존하던 연결은 재시작 후 갱신이 필요하다.
- 데몬은 기본적으로 루프백/로컬 소켓에만 바인딩하며, 외부 노출은 명시적 옵트인이 없으면 거부된다. 재시작도 이 로컬 우선 제약을 그대로 따른다.

## provenance
재시작 골격(중지 → 시작)은 두 곳에 존재한다. 데몬 자체(`daemon/src/main.rs`의 restart 처리: 중지 시도 후 `run_daemon` 재기동, 토큰 회전·로컬 바인딩 강제 포함)와, 사용자 진입점인 CLI(`cli/src/daemon.rs`의 restart: `cmd_stop` 후 `cmd_start`). 새 데몬의 준비 여부는 실행 기록 파일과 소켓 응답 프로브로 동기 확인한다.

## 미확정 (OPEN)
- [ ] OPEN: CLI 경로의 재시작(중지+시작 분리)과 데몬 자체 restart 서브커맨드(한 프로세스 내 중지 후 재기동)가 어느 상황에 각각 쓰이는지, 두 경로의 책임 분담이 의도된 것인지 owner 확인 필요.
- [ ] OPEN: 새 데몬이 "시작 중"으로만 보고된 채 끝나는 경우의 후속 확인 책임(호출자가 status로 재확인하는지)이 정책으로 명시되어 있는지 확인 필요.
