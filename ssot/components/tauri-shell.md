---
id: component.tauri-shell
kind: SystemComponent
title: Tauri 네이티브 셸
definition: 데스크탑 앱의 네이티브 껍데기. 앱이 켜질 때 로컬 데몬(clawketd)을 자식 프로세스로 띄우고, 데몬이 살아있는지 한 번 핑으로 확인하며, 데몬 인증에 필요한 세션 토큰을 디스크에서 읽어 화면(렌더러)에 건네는 역할을 맡는다.
purpose: 데스크탑 앱 실행 시 로컬 데몬을 자동 기동하고 헬스 핑으로 생존을 확인하며, 데몬 TCP 인증 토큰을 렌더러에 안전하게 노출한다.
realizedBy: []
implementedIn:
  - desktop/apps/desktop/src-tauri/src/lib.rs
  - desktop/apps/desktop/src-tauri/src/daemon.rs
  - desktop/apps/desktop/src-tauri/src/socket.rs
  - desktop/apps/desktop/src-tauri/src/token.rs
  - desktop/apps/desktop/src-tauri/Cargo.toml
dependsOn:
  - component.daemon
  - component.renderer
consumesApi:
  - endpoint.daemon-health
providesApi: []
integratesWith: []
impacts:
  - component.desktop-app
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
---

## 책임

Tauri 셸은 데스크탑 앱을 띄울 때 가장 먼저 동작하는 네이티브 계층이다. 세 가지 일을 책임진다.

첫째, 앱이 시작되면 로컬 데몬을 자식 프로세스로 띄운다. 데몬 실행 파일은 환경변수로 지정한 경로 → 플러그인 설치 위치(앱 바이너리 옆, 또는 그 상위의 데몬 설치 폴더) → 시스템 PATH 순으로 찾는다. 이미 데몬이 돌고 있어 두 번째 인스턴스가 거부되더라도 셸은 이를 오류로 보지 않고, 다음 단계의 헬스 핑으로 실제 생존을 판단한다. 즉 "내가 띄운 데몬이냐"가 아니라 "데몬이 살아있느냐"를 기준으로 삼는다.

둘째, 데몬이 자리를 잡을 시간을 잠깐 준 뒤 헬스 확인을 한 번 보낸다. 이 확인은 외부에 열린 포트가 아니라 로컬 유닉스 소켓을 통해 이뤄지며, 응답 여부를 앱 로그에 남긴다. 핑은 화면 렌더링과 별개의 비동기 경로에서 돌기 때문에 데몬 기동을 기다리느라 창이 멈추지 않는다.

셋째, 데몬이 TCP로 들어오는 요청을 인증할 때 쓰는 회전형 세션 토큰을 디스크에서 읽어 화면(렌더러)에 건넨다. 렌더러는 브라우저 방식으로 데몬과 통신하므로 매 요청에 이 토큰을 실어야 하는데, 유닉스 소켓 경로는 인증이 면제되는 반면 TCP 경로는 토큰이 필요하다. 셸은 이 토큰을 약속된 캐시 위치에서 찾아 읽고, 비어 있거나 못 읽으면 그 사유를 그대로 화면에 돌려준다.

## 경계와 의존

이 셸은 데스크탑 앱(`component.desktop-app`) 안에서만 존재하는 네이티브 부분이다. 화면을 그리는 일은 하지 않고, 그 일은 같은 앱 안의 렌더러(`component.renderer`)가 맡는다. 셸과 렌더러는 한 앱의 두 층으로, 셸이 네이티브 기능(프로세스 기동·토큰 읽기)을 명령 형태로 노출하면 렌더러가 호출한다.

셸이 의존하는 외부 대상은 로컬 데몬(`component.daemon`) 하나다. 데몬을 띄우고, 데몬의 헬스 확인(`endpoint.daemon-health`)을 부르며, 데몬이 발급한 토큰 파일을 읽는다. 데몬을 찾는 경로 규칙과 토큰·소켓 위치 규칙은 CLI 쪽 규칙과 같은 약속을 따르도록 맞춰져 있어, 누가 데몬을 띄웠든 동일한 위치에서 만난다.

## 통신 패턴

셸은 데몬과 두 가지 경로로 만난다. 하나는 프로세스 기동(자식 프로세스로 데몬 실행 파일을 띄움)이고, 다른 하나는 로컬 유닉스 소켓을 통한 단발성 헬스 핑이다. 핑은 TCP 폴백 없이 소켓만 쓰며, 짧은 타임아웃 안에 응답이 없으면 실패로 본다. 토큰은 네트워크가 아니라 디스크 파일 읽기로 가져온다. 렌더러와는 Tauri의 명령 브리지를 통해 토큰 읽기 기능을 노출하는 식으로 소통한다.

## 하위 서브패키지 (책임 단위)

- 진입·생명주기: 앱 빌드와 셋업 단계에서 데몬 기동·핑을 순서대로 엮는 부분.
- 데몬 기동: 데몬 실행 파일을 환경변수·설치 위치·PATH 순으로 찾아 자식 프로세스로 띄우는 부분.
- 소켓 핑: 유닉스 소켓으로 헬스 확인을 보내는 최소 HTTP 클라이언트.
- 토큰 읽기: 데몬 세션 토큰을 캐시 위치에서 읽어 렌더러에 노출하는 명령.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 기동 실패·핑 실패 시 화면에 사용자 안내를 띄우는지(현재는 로그만 남기는 것으로 보임) 확인 필요
- [ ] OPEN: 앱 종료 시 자식 데몬을 함께 정리하는 정책 여부 확인 필요
