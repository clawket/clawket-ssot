---
id: endpoint.cli-status
kind: Endpoint
title: 데몬 상태 확인 (clawketd status)
definition: 로컬 데몬이 지금 돌고 있는지·정상 응답하는지·어떤 데이터 스키마 버전을 쓰는지를 한눈에 보고하는 진단 명령이다.
realizedBy:
  - component.daemon
  - component.cli
implementedIn:
  - daemon/src/main.rs
  - cli/src/daemon.rs
relatesTo:
  - to: endpoint.cli-daemon
    type: belongs-to
    note: clawket daemon 명령군의 status 하위 동작이다.
  - to: endpoint.daemon-health
    type: reads
    note: 살아 있을 때 헬스 엔드포인트를 찔러 가동시간·스키마 버전을 읽는다.
governedBy: []
impacts: []
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
"데몬이 제대로 떠 있나?"를 점검하는 명령이다. 사용자가 직접 확인할 때뿐 아니라, 재시작·업데이트·설치 검증 같은 자동 흐름이 "다음 단계로 가도 되는지"를 판단하는 신호로도 쓰인다. 단순히 프로세스 존재 여부만 보지 않고 실제로 요청에 응답하는지(healthy)까지 확인해, 죽었지만 흔적만 남은 좀비 상태를 살아 있다고 오판하지 않는다.

## 요청 / 응답
- 보내는 것: 추가 입력 없는 단일 조회 지시.
- 돌려주는 것: 데몬의 현재 상태를 담은 구조화된 보고. 의미 단위로는 다음을 포함한다.
  - 프로세스 식별 정보와 접속 포트
  - 로컬 소켓 경로
  - 살아 있는지(alive) — 식별된 프로세스가 실제 존재하는가
  - 정상 응답하는지(healthy) — 헬스 신호에 응답하는가
  - 가동 시간(uptime) — 살아서 응답할 때 헬스 신호에서 읽음
  - 데이터 스키마 버전 — 응답하면 헬스에서, 멈춰 있으면 로컬 DB 파일에서 직접 읽어 보고
  - 실행 정보를 기록한 파일들의 위치
- 결과는 사람이 읽기 좋게 정리된 형태로 출력된다.

## 권한 / 제약
- 로컬 전용 진단 명령이다. 외부 원격 호출이 아니라 같은 머신에서 데몬 상태를 들여다보는 운영 명령이다.
- 데몬이 살아 있지 않으면 비정상 종료 코드(실패)로 끝난다. 자동화 스크립트가 "데몬 없음"을 명확히 분기할 수 있게 하기 위함이다.
- 데몬이 멈춰 있어도 스키마 버전만큼은 로컬 DB 파일에서 직접 읽어 보고하므로, 종료 상태에서도 "어떤 버전의 데이터인지"를 알 수 있다.

## provenance
상태 수집·보고의 본체는 데몬 자체(`daemon/src/main.rs`의 `status_daemon`, 헬스 필드 읽기 `fetch_health_fields`, 헬스 응답 구조 `Health`). 사용자 진입점과 종료 코드 전파는 CLI 쪽(`cli/src/daemon.rs`의 `cmd_status`). 살아 있을 때의 가동시간·스키마 버전은 데몬의 헬스 엔드포인트에서, 멈춰 있을 때의 스키마 버전은 로컬 DB 파일에서 읽는다.

## 미확정 (OPEN)
- [ ] OPEN: 보고에 담기는 항목 집합이 출력 형식(예: JSON 키 구성)으로 외부 계약화되어 다른 도구가 파싱에 의존하는지, 아니면 사람이 읽는 용도에 한정되는지 owner 확인 필요.
