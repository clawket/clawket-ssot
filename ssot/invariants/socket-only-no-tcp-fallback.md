---
id: invariant.socket-only-no-tcp-fallback
kind: Invariant
title: 셸의 헬스 핑은 Unix 소켓 전용·토큰 없음, 토큰 인증은 렌더러의 TCP 경로에만
definition: "데스크탑 네이티브 셸이 데몬 생존을 확인할 때는 Unix 소켓으로만 GET /health를 보내고 TCP 폴백도 토큰도 싣지 않는다(소켓 경로는 인증 면제). 토큰을 제시하는 쪽은 TCP로 통신하는 렌더러뿐이며, 모든 TCP 요청에 세션 토큰 헤더를 붙인다."
governs:
  - component.tauri-shell
  - integration.unix-socket-health
  - integration.session-token
  - endpoint.daemon-health
  - endpoint.cmd-read-token
decidedBy: []
implementedIn:
  - desktop/apps/desktop/src-tauri/src/socket.rs
  - desktop/apps/desktop/src-tauri/src/token.rs
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
relatesTo:
  - to: integration.unix-socket-health
    type: relates-to
    note: 헬스 핑을 소켓 전용·무토큰으로 제한한다
  - to: integration.session-token
    type: relates-to
    note: 토큰 제시 책임을 렌더러의 TCP 경로로 한정한다
  - to: invariant.daemon-liveness-socket-probe
    type: relates-to
    note: 소켓 핑으로 데몬 생존을 판정하는 같은 설계의 다른 표현
tags: [status:active, "type:invariant"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 OPEN. -->

## 제약

데스크탑 앱은 데몬과 두 갈래 경로로 대화하는데, 두 경로의 규칙이 다르다.

첫째, 네이티브 셸(앱 본체)이 "데몬이 살아 있나?"를 확인할 때는 오직 Unix 소켓으로만 헬스 확인 요청을 보낸다. 소켓이 안 되면 TCP로 우회하는 폴백은 없고, 인증 토큰도 싣지 않는다 — 로컬 소켓 경로는 같은 사용자만 접근 가능하므로 인증이 면제되기 때문이다.

둘째, 화면을 그리는 렌더러(브라우저형 프런트엔드)는 TCP로 데몬과 통신하므로, 데몬이 발급한 세션 토큰을 매 요청 헤더에 반드시 붙여 자신을 증명해야 한다. 즉 "토큰을 제시하는 쪽"과 "소켓으로 핑만 보내는 쪽"이 명확히 분리되어 있다.

## 깨지면 무슨 일이 일어나나

헬스 핑이 TCP로 폴백한다면, 소켓이 죽었는데도 TCP로 우연히 응답이 와서 "살아 있다"고 오판하거나, 인증이 필요한 TCP 경로에 토큰 없이 핑을 보내 불필요한 인증 실패가 나는 모순이 생긴다. 반대로 렌더러가 토큰을 빠뜨리면 정당한 사용자조차 TCP 요청이 거부된다. 경로별 규칙을 분리해 두면 "생존 확인"과 "인증된 데이터 접근"의 책임이 섞이지 않아, 로컬 우선 보안 모델(외부 유출 없음, 같은 사용자만 접근)이 깔끔하게 유지된다.

## 코드에서 어떻게 강제되나

소켓 클라이언트(`apps/desktop/src-tauri/src/socket.rs`)는 모듈 주석부터 "소켓 전용, TCP 폴백 없음, 토큰 로딩 없음"을 CLI 불변식과 동일하게 못 박고, 실제로 XDG 캐시의 데몬 소켓에 GET /health만 보낸다. 토큰 리더(`apps/desktop/src-tauri/src/token.rs`)는 데몬이 캐시에 쓴 세션 토큰 파일을 읽어 렌더러에 넘기는 역할이며, 주석에서 "Unix 소켓 경로는 인증 면제이지만 렌더러는 TCP로 통신하므로 매 요청에 이 토큰을 제시해야 한다"고 명시한다. 두 파일의 역할 분리 자체가 이 불변식의 구현이다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 소켓 전용·무폴백 설계 결정 기록
