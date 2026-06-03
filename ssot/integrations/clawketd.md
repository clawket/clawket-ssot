---
id: integration.clawketd
kind: Integration
title: CLI ↔ clawketd 데몬 연동
purpose: "CLI는 상태를 스스로 보관하지 않는다 — 모든 엔티티 읽기/쓰기를 로컬 데몬에 위임해, 여러 CLI 호출과 웹·MCP가 같은 단일 저장소를 공유하게 만든다."
definition: "clawket CLI가 로컬에서 도는 clawketd 데몬에게 모든 작업을 HTTP로 보내는 내부 연동. 통신은 로컬 유닉스 소켓 위의 HTTP로 이뤄지고, 데몬이 없으면 CLI가 알아서 데몬을 띄운다."
integratesWith:
  - component.cli
  - component.daemon
  - component.daemon-client
  - component.daemon-supervisor
implementedIn:
  - cli/src/client.rs
  - cli/src/daemon_autostart.rs
  - cli/src/mcp.rs
impacts:
  - endpoint.cli-daemon
  - endpoint.cli-mcp
  - endpoint.daemon-health
relatesTo:
  - to: component.daemon
    type: depends-on
    note: "CLI는 데몬이 살아 있어야 어떤 명령도 끝낼 수 있다 — 데몬이 모든 상태의 주인이다."
  - to: integration.unix-socket-health
    type: relates-to
    note: "이 연동의 생존 확인은 유닉스 소켓 접속 가능 여부로 판정한다."
governedBy:
  - invariant.daemon-liveness-socket-probe
  - invariant.daemon-flock-single-instance
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

clawket CLI 바이너리는 자기 안에 데이터를 들고 있지 않다. 프로젝트·플랜·태스크 같은 모든 엔티티의 읽기와 쓰기는 같은 머신에서 도는 별도 프로세스인 clawketd 데몬에게 맡긴다. CLI와 데몬은 같은 컴퓨터 안에서만 대화하며, 그 통로는 로컬 유닉스 소켓(파일 형태의 로컬 전용 연결구) 위에 얹은 HTTP다. 데몬은 같은 내용을 TCP(기본 localhost:19400)로도 열어 두지만, CLI 자신은 유닉스 소켓을 통해 붙는다. 어느 쪽이든 연결은 이 컴퓨터 밖으로 나가지 않는다.

이 위임 구조 덕분에 동시에 떠 있는 여러 CLI 호출, 웹 대시보드, MCP 서버가 전부 같은 데몬·같은 저장소를 바라본다. 상태가 한 곳에만 있으므로 누가 무엇을 바꿨는지 서로 어긋나지 않는다.

## 구현 위치 (provenance)

CLI가 데몬에 붙는 HTTP 클라이언트는 `cli/src/client.rs`에 있다. 여기서 유닉스 소켓 경로로 연결을 만들고, 모든 요청 URL을 `http://localhost…` 형태로 보내며, 연결에 실패하면 "clawketd에 붙지 못했다 — 실행 중인가? (`clawket daemon start`)"라는 안내를 띄운다. 즉 CLI의 거의 모든 동작이 이 한 클라이언트를 거친다.

데몬이 떠 있지 않을 때 CLI가 스스로 데몬을 띄우는 로직은 `cli/src/daemon_autostart.rs`에 있다. 순서는 (1) 소켓이 이미 살아 있으면 그대로 진행, (2) 환경변수로 자동기동을 끈 경우 건너뜀, (3) 파일 잠금을 잡아 여러 CLI가 동시에 데몬을 띄우려는 경쟁을 막고, (4) 잠금을 잡은 김에 소켓을 다시 확인해 다른 호출이 이미 띄웠으면 그대로 사용, (5) 그래도 없으면 데몬을 백그라운드로 분리 실행(독립 프로세스 그룹, 로그는 상태 디렉터리 파일로)한다. 기동 후 최대 8초까지 소켓이 열리길 기다리되, 시간이 지나도 하드 실패하지 않고 경고만 낸다 — 첫 실행 시 임베딩 모델 로딩으로 8초를 넘길 수 있기 때문이다.

이 자동기동은 특히 장수명 프로세스인 `clawket mcp`에서 중요하다(`cli/src/mcp.rs`). MCP 서버는 시작할 때 데몬을 먼저 보장하고, 데몬을 wait하지 않는 구조라 좀비 프로세스를 막기 위해 이중 포크로 데몬을 init의 자식으로 떼어낸다.

내부적으로 이 연동을 사용하는 컴포넌트는 데몬 클라이언트(`component.daemon-client`)와 자동기동 감독자(`component.daemon-supervisor`)이며, 상대편은 데몬 본체(`component.daemon`)다.

## 불변식

- 데몬의 생존은 항상 "유닉스 소켓에 접속이 되는가"로 판정한다(`invariant.daemon-liveness-socket-probe`). PID 파일은 보조 신호일 뿐, 최종 판정 기준이 아니다.
- 동시에 들어온 여러 CLI 호출이 데몬을 중복 기동하지 못하도록, 자동기동은 파일 잠금(flock)으로 직렬화한다(`invariant.daemon-flock-single-instance`). 잠금을 잡지 못한 호출은 띄우지 않고 소켓이 열리기를 기다린다.
- 데몬에 붙지 못하면 CLI는 명령을 완료할 수 없고, 사용자에게 데몬 기동 안내를 돌려준다(하드 의존).

## 영향 범위

이 연동이 끊기거나 동작이 바뀌면 데몬 수명주기 명령(`endpoint.cli-daemon`)과 MCP 서버 기동(`endpoint.cli-mcp`)이 직접 영향을 받고, 설치 후 헬스 확인(`endpoint.daemon-health`)으로 생존을 점검한다. 더 넓게는 데몬을 거치는 모든 CLI 명령이 이 연동에 의존한다.

## 미확정 (OPEN)
- [ ] OPEN: 자동기동 비활성 환경변수(`CLAWKET_NO_AUTOSPAWN`)·데몬 바이너리 지정(`CLAWKET_DAEMON_BIN`)을 별도 노드로 다룰지 여부
- [ ] OPEN: TCP(localhost:19400) 경로를 별도 연동 노드로 분리할지 여부
