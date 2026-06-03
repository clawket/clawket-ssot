---
id: component.daemon-client
kind: SystemComponent
title: 데몬 클라이언트 (CLI→clawketd HTTP-over-UDS)
definition: clawket CLI가 로컬 clawketd 데몬과 대화하기 위해 쓰는 내부 통신 계층으로, 유닉스 도메인 소켓 위에서 HTTP를 주고받으며 단순 조회·JSON 요청·원시 바이트·끝나지 않는 이벤트 스트림(SSE)까지 네 갈래 요청 방식을 한 곳에서 제공한다.
purpose: CLI의 거의 모든 명령은 자체적으로 데이터를 들고 있지 않고 데몬에 물어보거나 변경을 요청해야 한다. 이 클라이언트는 그 "데몬에게 말 거는 방식"을 한 군데로 모아, 명령마다 소켓 연결·HTTP 구성·오류 해석을 다시 작성하지 않도록 한다. 특히 사람이 읽을 수 있는 실패 메시지(데몬이 안 떠 있으면 "clawketd가 실행 중인가요? clawket daemon start" 안내)와, 무한히 흐르는 이벤트 스트림을 한 줄씩 점진적으로 받아 처리하는 능력을 표준화한다.
realizedBy:
  - component.cli
implementedIn:
  - cli/src/client.rs
dependsOn:
  - component.cli
  - component.paths
  - component.daemon
consumesApi:
  - endpoint.daemon-health
  - endpoint.daemon-events
  - endpoint.daemon-search
providesApi: []
integratesWith: []
impacts:
  - platform.local-first
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
---

## 책임

이 컴포넌트는 clawket CLI 안에서 "데몬과 통신하는 단일 창구" 역할을 맡는다. 사용자가 `clawket task list`, `clawket dashboard`, `clawket watch` 같은 명령을 실행하면, 그 명령들은 직접 파일이나 데이터베이스를 건드리지 않고 이 창구를 통해 로컬 clawketd 데몬에 요청을 보낸다.

네 가지 통신 방식을 제공한다.

- **단순 조회**: 데몬에서 JSON 한 덩어리를 받아오는 가장 흔한 경우(목록·상세 조회 등).
- **원시 바이트 조회**: JSON이 아닌 페이로드를 그대로 받는 경우. 대표적으로 유한하게 끝나는 리플레이용 이벤트 스트림을 통째로 받을 때 쓴다.
- **변경 요청**: 생성·수정·삭제처럼 본문(JSON)을 실어 보내는 요청. 성공/실패를 가려주는 형태와, 상태 코드와 본문을 그대로 돌려줘 호출 측이 직접 판단하게 하는 형태 두 가지가 있다(예: 잠금 충돌 시 누가 점유 중인지 같은 구조화된 상세를 들여다봐야 할 때).
- **실시간 스트림 구독**: 끝나지 않고 계속 흐르는 이벤트 스트림(SSE)을 한 건씩 받아 즉시 처리한다. `clawket watch`가 데몬의 무한 이벤트 스트림을 화면에 점진적으로 그릴 때 쓰인다.

오류를 사람이 이해할 수 있게 번역하는 것도 이 컴포넌트의 책임이다. 연결 자체가 실패하면 "데몬이 떠 있는지" 안내를 붙이고, 데몬이 실패 응답을 주면 그 안의 오류 메시지를 꺼내 보여준다.

## 경계와 의존

이 컴포넌트는 CLI 바이너리(`component.cli`)의 일부로 컴파일되는 내부 모듈이며, 별도 프로세스나 외부 서비스가 아니다. 통신 상대는 같은 기기에서 도는 clawketd 데몬(`component.daemon`) 하나뿐이고, 네트워크로 외부에 나가지 않는다 — 이는 모든 데이터가 로컬에 머문다는 local-first 동작(`platform.local-first`)을 떠받친다.

연결할 소켓의 경로는 스스로 계산하지 않고 경로 해석 컴포넌트(`component.paths`)에 위임한다. 데몬이 노출하는 HTTP API 가운데 헬스 확인(`endpoint.daemon-health`), 이벤트 스트림(`endpoint.daemon-events`), 검색(`endpoint.daemon-search`) 등을 호출하는 쪽이며, 자신이 외부에 제공하는 API는 없다(상위 CLI 명령들이 이 모듈을 라이브러리처럼 호출한다).

## 통신 패턴

통신은 "유닉스 도메인 소켓 위의 HTTP"라는 다소 특이한 형태다. 일반적인 TCP 포트가 아니라 파일시스템 상의 소켓 경로로 데몬에 연결하므로, 같은 사용자 계정 안에서만 닿을 수 있고 외부 네트워크에 노출되지 않는다. 요청은 보통의 HTTP(GET/POST 등)로 보내되 호스트는 명목상 로컬을 가리킨다.

실시간 이벤트는 Server-Sent Events 규약을 따라 흐른다. 무한히 이어지는 스트림이라 응답 전체를 기다려 한 번에 받는 방식으로는 처리할 수 없고, 도착하는 대로 줄 단위로 모아 하나의 이벤트가 완성될 때마다 호출 측에 전달한다.

## 하위 서브패키지 (책임 단위)

CLI 안의 단일 모듈로, 별도의 하위 패키지로 다시 쪼개지지 않는다. 요청 방식별로 함수가 나뉘어 있을 뿐 책임 단위는 "데몬 통신"으로 하나다.

## 미확정 (OPEN)
- [ ] OPEN: 인벤토리상 함께 언급된 데스크탑 앱의 데몬 API 클라이언트(apps/desktop의 통신 계층)와 이 CLI 클라이언트가 동일한 통신 규약·엔드포인트를 공유하는지(설계 의도상 한 쌍인지) 코드만으로는 단정하지 않았다 — 같은 데몬을 향하지만 서로 독립 구현으로 보인다.
