---
id: endpoint.cli-start
kind: Endpoint
title: clawketd start (데몬 포그라운드 실행 — 기본 명령)
definition: 로컬 데몬을 포그라운드로 띄우는 기본 명령으로, 바인딩 포트·주소·DB 경로·지표 노출 여부를 인자로 받아 HTTP·유닉스소켓 리스너와 SQLite 저장소를 연다.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/config.rs
  - daemon/src/main.rs
relatesTo:
  - { to: concept.knowledge, type: mutates, note: "기동 시 임베딩이 없는 과거 태스크를 백그라운드로 백필해 검색 가능하게 만든다" }
  - { to: concept.task, type: reads, note: "백필 대상(임베딩 누락 태스크)을 훑어 임베딩을 채운다" }
  - { to: endpoint.cli-daemon, type: backed-by, note: "clawket daemon start/restart 가 최종적으로 이 실행 경로를 띄운다" }
governedBy:
  - invariant.public-bind-not-allowed
  - invariant.local-only-loopback
  - invariant.migration-gate
  - invariant.schema-version-discipline
impacts: []
consumedBy:
  - endpoint.cli-daemon
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

클로켓의 로컬 데몬 프로세스를 직접 띄우는 진입점이다. 하위 명령을 주지 않으면 이 실행이 기본으로 동작하며, 띄운 자리에서 포그라운드로 머문다(상위 관리자나 사용자 명령이 이 프로세스를 감싼다). 데몬은 모든 상태를 담은 SQLite 저장소를 열고, 같은 머신의 CLI·웹·MCP가 붙을 HTTP/유닉스소켓 리스너를 연다.

## 요청 / 응답

명령 인자로 다음을 받는다 — 바인딩 포트(기본 19400, 이미 사용 중이면 +1씩 최대 20번까지 다음 빈 포트로 자동 이동, 0을 주면 OS가 임의 포트 배정), 바인딩 주소(기본 루프백 127.0.0.1), DB 경로 오버라이드(기본 경로 대신 다른 SQLite 파일 지정), 그리고 지표 엔드포인트 노출 여부(환경변수로도 켤 수 있다).

기동하면 저장소를 열어 보류 중인 스키마 마이그레이션을 반영하고, 실제로 바인딩된 포트·PID·소켓 경로·인증 토큰을 파일로 기록해 다른 클라이언트가 데몬을 찾고 인증할 수 있게 한다(토큰은 재시작마다 새로 발급). 유닉스소켓은 소유자 전용 권한으로 잠그고, 같은 머신의 신뢰 경로로 본다. 기동 직후에는 임베딩이 빠진 과거 태스크를 백그라운드로 훑어 채우는 백필과 활동 로그 정리 루프를 분리 실행해, 기동 지연이 HTTP 바인딩 시간에 묶이지 않게 한다. 종료 신호를 받으면 진행 중 요청에 짧은 유예를 준 뒤, 오래 붙어 있는 연결이 종료를 막으면 강제 종료하고 기록 파일들을 정리한다.

## 권한 / 제약

이 데몬은 로컬 전용이며 SQLite 저장소는 여러 사용자에게 노출되도록 설계되지 않았다. 따라서 루프백이 아닌 주소로 바인딩하려는 시도는 거부하며, 운영자가 명시적으로 공개 바인딩을 허용하는 환경변수를 켠 경우에만 예외다. TCP 리스너에는 발급된 토큰 기반 인증이 붙고, 유닉스소켓은 로컬=신뢰로 보아 인증을 면제한다. 스키마 마이그레이션이 진행 중인 동안 변경 요청은 일시적으로 거부(마이그레이션 게이트)되며, 스키마 버전은 별도 규율로 관리된다.

## provenance

명령 인자 정의는 daemon/src/config.rs(StartArgs)에, 기동·바인딩·종료·백필 로직은 daemon/src/main.rs(run_daemon 등)에 있다. 사용자·관리자 관점의 데몬 기동/중지/재시작 명령은 endpoint.cli-daemon 에 정리돼 있다. 공개 바인딩 거부·루프백 전용·마이그레이션 게이트·스키마 버전 규율은 각각 invariant.public-bind-not-allowed, invariant.local-only-loopback, invariant.migration-gate, invariant.schema-version-discipline 으로 못 박혀 있다.

## 미확정 (OPEN)
- [ ] OPEN: 종료 유예 시간·포트 자동이동 횟수 등 운영 파라미터가 환경변수로만 조정되는지 설정 파일로도 노출되는지, 지표 엔드포인트의 노출 표면 범위를 owner 확인.
