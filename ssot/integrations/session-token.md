---
id: integration.session-token
kind: Integration
title: 세션 토큰 기반 데몬 인증
purpose: 데스크탑의 화면(렌더러)은 브라우저와 같은 방식으로 동작해 사용자 캐시 파일을 직접 읽을 수 없다. 그러나 데몬의 데이터 통신은 인증을 요구한다. 그래서 파일을 읽을 수 있는 네이티브 쪽이 토큰을 대신 읽어 화면에 건네주고, 화면은 그 토큰을 매 요청에 붙여 신원을 증명한다.
definition: 데몬이 시작할 때 사용자 캐시에 써 둔 세션 토큰을, 데스크탑의 네이티브 명령(read_token)이 읽어 렌더러에 전달하고, 렌더러가 모든 데몬 TCP 요청의 인증 헤더(X-Clawket-Token)로 제시하는 인증 연동이다.
integratesWith:
  - component.desktop-app
  - component.daemon-client
  - component.daemon
implementedIn:
  - desktop/apps/desktop/src-tauri/src/token.rs
  - desktop/apps/desktop/src/data/api.ts
impacts:
  - screen.daemon-offline-state
consumesApi:
  - endpoint.cmd-read-token
governedBy:
  - invariant.tcp-auth
  - invariant.auth-cookie-token
relatesTo:
  - to: integration.clawketd-daemon
    type: relates-to
    note: 인증 대상 데몬은 이 연동으로 띄운 바로 그 데몬이다.
  - to: integration.unix-socket-health
    type: relates-to
    note: 생존 확인(소켓)은 인증 면제, 데이터 통신(TCP)은 토큰 필요로 경로가 갈린다.
  - to: endpoint.cmd-read-token
    type: depends-on
    note: 토큰을 파일에서 읽어 화면에 넘기는 네이티브 명령.
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

데몬은 TCP로 들어오는 데이터 요청에 인증을 요구한다. 인증 수단은 데몬이 시작할 때 발급해 사용자 캐시 영역(XDG 캐시) 아래 토큰 파일에 적어 두는 세션 토큰이다. 데몬 쪽 미들웨어는 이 토큰을 인증 헤더(X-Clawket-Token) 값이나 쿠키로 받아들인다.

문제는 데스크탑의 화면(렌더러)이 브라우저처럼 동작해 디스크의 캐시 파일을 직접 읽을 권한이 없다는 점이다. 그래서 파일 접근이 가능한 네이티브 쪽이 다리를 놓는다. 네이티브에 노출된 명령(read_token)이 토큰 파일을 읽어 앞뒤 공백을 정리한 문자열로 화면에 돌려준다. 파일이 없거나 비어 있으면 빈 값 대신 오류 메시지를 돌려주어, 화면이 인증 실패 상황을 사용자에게 드러낼 수 있게 한다.

화면 쪽 HTTP 클라이언트는 이 토큰을 한 번 받아 보관해 두고, 데몬으로 가는 모든 요청에 인증 헤더로 붙인다. 인증이 거부되면(401 응답) 보관해 둔 토큰을 버리고, 다음 요청 때 다시 읽어 오도록 한다 — 토큰이 갱신되었을 가능성에 대비한 처리다. 또한 Tauri 런타임이 없는 순수 웹 개발 모드에서는 네이티브 명령을 쓸 수 없으므로 빈 토큰으로 떨어지고, 그 경우 데몬이 요청을 거부해 인증 오류가 화면에 드러난다.

토큰 파일의 위치는 환경 변수(전용 캐시 디렉터리 지정 → XDG 캐시 → 홈의 .cache) 순으로 결정되며, 데몬 쪽 캐시 경로 규칙과 동일하게 맞춰져 있다.

## 구현 위치 (provenance)

- `desktop/apps/desktop/src-tauri/src/token.rs` — 토큰 파일 경로 계산과, 파일을 읽어 정리한 문자열(또는 오류)을 화면으로 돌려주는 네이티브 명령이 여기 있다. 코드 주석에 "유닉스 소켓 경로는 인증 면제지만 렌더러는 TCP로 말하므로 매 요청에 토큰을 제시해야 한다"는 설계 근거가 명시되어 있다.
- `desktop/apps/desktop/src/data/api.ts` — 토큰을 읽어 보관하고(캐싱), 모든 요청에 인증 헤더로 붙이며, 인증 거부 시 토큰을 무효화해 재요청하는 클라이언트 로직이 여기 있다. 기본 접속 주소는 루프백 19400이다.

이 연동을 사용하는 내부 컴포넌트는 데스크탑 앱(component.desktop-app)과 HTTP 클라이언트(component.daemon-client)이며, 인증 상대는 데몬 본체(component.daemon)다.

## 불변식

- 데몬의 TCP 요청은 인증을 거쳐야 한다(invariant.tcp-auth). 화면은 토큰 없이 데이터 통신을 할 수 없다.
- 인증 토큰은 헤더 또는 쿠키 형태로 제시된다(invariant.auth-cookie-token). 데스크탑은 그중 헤더 방식을 사용한다.

## 영향 범위

- 토큰을 읽지 못하거나 인증이 거부되면 데이터 요청이 전부 실패하므로, 사용자는 사실상 데몬을 못 쓰는 상태가 된다. 이 실패는 오프라인 상태 화면(screen.daemon-offline-state)으로 이어질 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 재시작으로 토큰이 회전(rotate)될 때, 화면이 401을 받아 토큰을 다시 읽어 복구하는 흐름이 모든 진행 중 요청에 대해 매끄럽게 동작하는지(재시도 횟수·동시 요청 처리) 검증 필요.
