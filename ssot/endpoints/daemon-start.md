---
id: endpoint.daemon-start
kind: Endpoint
title: 데몬 백그라운드 기동 명령
definition: 로컬 데몬(clawketd)을 백그라운드로 띄우는 명령. 호출자를 붙잡지 않도록 완전히 분리(detach)해 띄우고, 잠깐 살아있는지 확인한 뒤 "준비됨"을 동기적으로 알려준다.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/daemon.rs
relatesTo:
  - to: component.daemon
    type: calls
    note: 실제 HTTP 서버 프로세스(clawketd)를 자식 프로세스로 띄운다.
  - to: endpoint.daemon-health
    type: reads
    note: 기동 직후 소켓 응답 여부로 "이미 살아있는지 / 준비됐는지"를 판정한다.
governedBy: []
impacts: []
consumedBy:
  - endpoint.hook-session-start
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

이 명령은 모든 작업 상태를 보관하는 로컬 데몬을 백그라운드로 띄운다. 데몬 본체는 HTTP 서버를 포그라운드로 도는 프로세스라, 만약 이 명령이 그 프로세스가 끝날 때까지 기다린다면 데몬을 동기적으로 호출하는 세션 시작 훅이 데몬이 사는 내내 멈춰버린다. 그래서 이 명령은 데몬을 띄우되 절대 기다리지 않도록 완전히 분리해 실행한다.

분리 방식은 표준 입력을 비우고, 출력·오류는 상태 디렉터리 아래 로그 파일에 이어 붙이며, 새 프로세스 그룹으로 띄워 호출자(예: 훅을 실행한 셸)에 전달된 중단 신호가 데몬으로 번지지 않게 한다. 또한 이중 fork 로 데몬을 init 프로세스에 재부모화해 좀비로 남지 않게 한다.

띄운 뒤에는 최대 5초까지 데몬이 살아났는지(소켓 응답) 폴링해 호출자에게 "준비됨"이라는 동기 신호를 준다. 5초 안에 확인되지 않으면 실패로 처리하지 않고 "기동 중"으로 둔다 — 훅을 타임아웃시키지 않기 위해서다.

## 요청 / 응답

입력 인자는 없다. 응답은 사람이 읽을 짧은 상태 문구다. 데몬이 이미 떠 있으면 "이미 실행 중(pid 포함)"을 알리고 새로 띄우지 않는다(멱등). 새로 띄웠고 살아남이 확인되면 준비됨을, 시간 안에 확인되지 않으면 기동 중을 알린다.

생존 판정은 "pid 파일이 있느냐"가 아니라 "데몬이 소켓에 응답하느냐"를 본다. 그래서 pid 만 남고 실제로는 죽은(좀비) 상태는 죽은 것으로 취급되어 깨끗하게 다시 띄워진다. pid 값은 메시지 표기용으로만 읽는다.

## 권한 / 제약

로컬 머신에서 데몬 프로세스를 띄우는 명령이라 외부로 나가는 동작이 없다. 데몬 바이너리를 찾을 수 없으면(경로에 없거나 환경변수 미설정) 기동에 실패하며, 그 사유를 안내한다.

## provenance

기동·분리·생존확인 로직 근거는 cli/src/daemon.rs 의 start 처리부. 세션 시작 시 이 명령을 동기 호출해 데몬을 보장하는 소비자는 SessionStart 훅이다.

## 미확정 (OPEN)
- [ ] 없음(현 범위에서 동작·제약 확정).
