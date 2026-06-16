---
id: endpoint.daemon-log
kind: Endpoint
title: clawket daemon log
definition: "로컬 데몬의 로그 파일을 최근 N줄만 보여주거나 -f로 실시간 추적(tail)하는 진단용 명령."
realizedBy: [component.cli, component.daemon-supervisor]
implementedIn: [cli/src/main.rs, cli/src/daemon.rs]
relatesTo:
  - { to: endpoint.daemon-start, type: relates-to, note: "로그가 없으면 먼저 데몬을 기동하라고 안내" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
사용자(또는 운영자)가 로컬 clawketd 데몬이 무슨 일을 했는지 들여다보고 싶을 때 쓰는 진단 명령이다. 데몬은 표준 상태 디렉터리(`~/.local/state/clawket/`)에 자신의 로그를 남기는데, 이 명령은 그 로그 파일의 끝부분을 화면에 보여주거나, 데몬이 동작하는 동안 새로 쌓이는 줄을 실시간으로 따라가며 보여준다. 데몬이 떠 있지 않거나 한 번도 실행되지 않아 로그 파일이 아직 없으면, "로그를 찾을 수 없다"는 안내와 함께 먼저 데몬을 시작하라는 힌트를 출력하고 정상 종료한다.

## 요청 / 응답
- 보낼 수 있는 것: 보고 싶은 최근 줄 수(기본 50줄), 실시간 따라가기 여부(켜면 데몬이 멈추거나 사용자가 중단할 때까지 새 줄을 계속 흘려보낸다).
- 돌려주는 것: 로그 파일의 마지막 부분 텍스트 그대로. 별도의 구조화된 데이터가 아니라 사람이 읽는 원시 로그 줄이다. 파일이 없으면 안내 메시지와 시작 힌트만 나온다.

## 권한 / 제약
로컬 머신에서 그 사용자 계정으로 실행하는 진단 명령이며, 외부 인증 개념은 없다(로컬 우선 동작). 로그 위치는 사용자 홈 기준으로 결정되므로 홈 경로를 알 수 없는 환경에서는 경로를 특정할 수 없어 실패한다. 실시간 추적과 단순 출력 모두 내부적으로 시스템의 `tail` 유틸리티에 의존하므로, 그 유틸리티가 없는 환경에서는 동작이 제한될 수 있다(확인 필요).

## provenance
명령 정의와 옵션은 CLI 진입점(`cli/src/main.rs`)의 daemon 하위명령에, 실제 로그 파일 경로 해석과 출력/추적 동작은 `cli/src/daemon.rs`에 있다. 제공 주체는 CLI 바이너리이며, 데몬 프로세스 수명·로그 파일을 다루는 책임은 데몬 감독 컴포넌트에 속한다.

## 미확정 (OPEN)
- [ ] OPEN: `tail` 외부 의존이 모든 지원 플랫폼(특히 비유닉스)에서 동일하게 동작하는지 확인 필요.
