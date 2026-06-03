---
id: decision.daemon-over-cli-only
kind: Decision
title: CLI-only 대신 상시 데몬 구조를 택한다
purpose: "한 번 실행하고 끝나는 단발 CLI만으로 구성할지, 아니면 백그라운드에 상시 떠 있는 데몬을 함께 둘지"
definition: "쓰기 직렬화·임베딩 모델 상주·웹 대시보드 호스팅·서브에이전트 훅 직렬화를 위해, CLI 단독 구조 대신 항상 떠 있는 로컬 데몬을 함께 운영한다."
relatesTo:
  - { to: component.daemon, type: governs, note: "이 결정이 데몬이라는 구성요소의 존재 근거다" }
  - { to: component.cli, type: relates-to, note: "CLI는 데몬에 위임하는 얇은 클라이언트로 남는다" }
  - { to: component.sqlite-storage, type: relates-to, note: "단일 writer 직렬화의 대상 저장소" }
  - { to: component.embedding-engine, type: relates-to, note: "데몬이 메모리에 붙들고 있는 임베딩 모델" }
  - { to: component.web-dashboard, type: relates-to, note: "데몬이 HTTP로 함께 서빙하는 대시보드" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

Clawket은 작업 관리 상태(프로젝트·플랜·태스크·지식·실행 이력)를 로컬에 보존하고, LLM 에이전트와 사람이 같은 상태를 읽고 쓰는 도구다. 가장 단순한 구현은 "명령을 한 번 실행하고 끝나는" 단발 CLI다. 그러나 이 방식만으로는 다음 네 가지 요구를 동시에 만족시키기 어려웠다.

- 여러 CLI 인스턴스(또는 병렬 서브에이전트의 훅)가 동시에 같은 데이터에 쓰려 할 때, 쓰기를 어딘가에서 직렬화해야 한다. 단발 프로세스끼리는 서로의 쓰기를 조율할 주체가 없어 락 충돌이 잦아진다.
- 지식 검색용 임베딩 모델(다국어 384차원 모델)을 매 호출마다 메모리에 새로 적재하면 한 번에 몇 초씩 걸린다. 명령 하나하나가 느려진다.
- 사람이 보는 웹 대시보드를 띄우려면 HTTP 서버가 어딘가 상주해야 한다.
- Claude Code에서 서브에이전트가 병렬로 떠 훅이 동시에 발동할 때, 이를 순서대로 받아 처리할 큐가 필요하다.

## 결정 (Decision)

CLI 단독 구조 대신, 백그라운드에 항상 떠 있는 로컬 데몬을 함께 운영하기로 했다. CLI는 데몬에 일을 위임하는 얇은 클라이언트로 남고, 데이터에 대한 모든 쓰기·검색·대시보드 서빙은 데몬이 단일 주체로 담당한다.

## 근거와 결과 (Consequences)

데몬을 둠으로써 네 가지 이점을 한 번에 얻는다.

1. 데이터베이스의 모든 쓰기를 데몬 한 곳에서 직렬화한다. 동시 읽기는 허용하되 쓰기는 한 줄로 모아 락 충돌을 없앤다. 이것이 데몬이 "단일 writer" 역할을 한다는 의미다.
2. 임베딩 모델을 데몬이 메모리에 계속 붙들고 있으므로, 개별 명령 호출은 약 10밀리초 수준으로 빠르게 유지된다. 모델을 매번 새로 적재하는 비용이 사라진다.
3. 데몬이 이미 떠 있으니 HTTP 서버를 붙여 웹 대시보드를 같은 데이터 연결 위에서 함께 서빙한다.
4. 병렬 서브에이전트의 훅 호출을 데몬이 큐로 받아 순서대로 처리한다.

대신 받아들인 트레이드오프는 두 가지다. 첫째, 데몬 프로세스 하나가 상시 떠 있어야 한다(메모리는 약 7MB 수준으로 가볍다). 둘째, 첫 세션에서 CLI·데몬 바이너리와 웹 번들을 한 번 내려받는 설치 과정이 필요하다. 이 비용을 감수할 가치가 있다고 판단했으며, CLI 단독 구조로는 위 이점들을 얻기 어렵다고 보았다.

이 결정은 데몬이라는 구성요소가 존재하는 근본 이유이며, CLI가 직접 데이터를 쓰지 않고 데몬에 위임하는 클라이언트 구조, 그리고 데몬이 검색·대시보드까지 떠안는 형태를 함께 규정한다.

<!-- provenance: clawket/BLOG.md:72-92 ("구현: 왜 데몬 구조인가" — 4개 이점·트레이드오프·~7MB·~10ms), clawket/clawket/README.md:15,39,66,222,404 (lightweight daemon / on-device 임베딩 / axum+rusqlite / 대시보드 포트 19400 / loopback-only) -->
