---
id: integration.unix-socket-health
kind: Integration
title: 유닉스 소켓 생존 확인(health probe)
purpose: 데몬을 띄우려 시도한 것만으로는 "쓸 수 있는 데몬이 있다"고 보장되지 않는다. 이미 떠 있던 데몬을 쓰는 경우도, 방금 띄운 데몬이 아직 준비 중인 경우도 있기 때문이다. 그래서 앱은 실제로 응답하는지를 직접 물어 확인한다.
definition: 데스크탑 셸이 시작 직후, 운영체제의 유닉스 도메인 소켓을 통해 데몬에게 생존 확인 요청(GET /health)을 보내 응답 여부로 데몬이 살아 있는지 판정하는 연동이다. TCP로 우회하지 않는 소켓 전용 경로다.
integratesWith:
  - component.desktop-app
  - component.daemon-supervisor
  - component.daemon
implementedIn:
  - desktop/apps/desktop/src-tauri/src/socket.rs
impacts:
  - screen.daemon-offline-state
consumesApi:
  - endpoint.daemon-health
governedBy:
  - invariant.daemon-liveness-socket-probe
  - invariant.socket-only-no-tcp-fallback
relatesTo:
  - to: integration.clawketd-daemon
    type: relates-to
    note: 데몬을 띄운 직후 이 확인 절차가 이어진다.
  - to: endpoint.daemon-health
    type: depends-on
    note: 확인 대상은 데몬의 헬스 엔드포인트다.
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

데스크탑 셸은 데몬이 정말로 응답하는지를 직접 확인한다. 확인 통로는 TCP 네트워크 포트가 아니라, 운영체제가 제공하는 로컬 전용 통로인 유닉스 도메인 소켓이다. 소켓 파일의 위치는 사용자 캐시 영역(XDG 캐시) 아래 정해진 경로로, 환경에 따라 기본값이 결정된다.

이 통로로 앱은 가장 단순한 HTTP 요청 하나(생존 확인용 GET /health)를 보내고, 돌아오는 응답의 상태 코드와 본문을 읽는다. 정상 응답(200)이면 "쓸 수 있는 데몬이 있다"고 판단한다. 요청에는 짧은 제한 시간(약 2초)이 걸려 있어, 데몬이 응답하지 않으면 무한정 기다리지 않고 시간 초과로 실패 처리한다.

중요한 설계 선택은 이 경로가 "소켓 전용"이라는 점이다. 소켓으로 닿지 못하면 TCP 포트로 다시 시도하는 폴백을 두지 않는다. 또한 이 소켓 경로는 토큰 인증이 면제되어 있어, 생존 확인만큼은 별도 인증 없이 가능하다(실제 데이터 통신은 TCP + 토큰으로 한다 — 별도 연동 노드 참고).

## 구현 위치 (provenance)

- `desktop/apps/desktop/src-tauri/src/socket.rs` — 기본 소켓 경로 계산, 소켓 연결, 생존 확인 요청 전송, 응답의 상태 코드·본문 파싱, 그리고 제한 시간 처리가 모두 여기 있다. CLI 쪽의 "소켓 전용, TCP 폴백 없음" 규칙을 그대로 따른다고 코드 주석에 명시되어 있다.

이 연동을 사용하는 내부 컴포넌트는 데스크탑 앱(component.desktop-app)과 그 안의 데몬 감독 모듈(component.daemon-supervisor)이며, 확인 대상은 데몬 본체(component.daemon)다. 이 절차는 앱 시작 시퀀스(lib.rs)에서 데몬을 띄운 뒤 잠깐의 대기 후 비동기로 호출된다.

## 불변식

- 살아 있음의 판단은 프로세스 생성 여부가 아니라 실제 응답 여부로 한다(invariant.daemon-liveness-socket-probe). 그래서 "방금 띄웠다"가 아니라 "응답이 온다"를 확인한다.
- 이 확인은 소켓 전용 경로이며 TCP로 우회하지 않는다(invariant.socket-only-no-tcp-fallback). 소켓으로 닿지 못하면 그대로 실패로 본다.

## 영향 범위

- 생존 확인이 실패하면(시간 초과 또는 비정상 응답) 앱은 데몬을 쓸 수 없는 상태로 간주하고, 오프라인 상태 화면(screen.daemon-offline-state)으로 사용자에게 알린다.

## 미확정 (OPEN)
- [ ] OPEN: 최초 1회 실패 후 재확인(재시도/폴링) 정책 — 현재 코드 경로에서는 시작 시 단발 확인만 보이고, 주기적 재확인은 HTTP 클라이언트의 health() 쪽 책임으로 분리되어 있을 가능성. 두 경로의 역할 분담 확정 필요.
