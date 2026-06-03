---
id: capability.self-update
kind: Capability
title: 최신 버전 설치·업데이트 확인
definition: 사용자가 Clawket의 최신 릴리스를 내려받아 설치하거나, 설치하지 않고 더 새 버전이 있는지만 확인하도록 돕는 기능. 실제 다운로드·설치는 로컬 데몬에 위임된다.
purpose: "사용자가 별도 설치 절차를 따로 밟지 않고도 Clawket의 CLI와 데몬을 최신 상태로 유지하거나, 업데이트가 필요한지 먼저 가볍게 확인할 수 있게 한다."
servesPersona: [persona.human-operator, persona.llm-coding-agent]
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.cli-update-versioncheck, type: calls, note: "설치 및 버전 확인은 update/version-check 명령으로 수행" }
  - { to: integration.github-releases, type: depends-on, note: "릴리스 산출물은 GitHub Releases에서 받는다" }
  - { to: component.daemon, type: depends-on, note: "다운로드·설치 처리는 데몬의 self-update 엔드포인트가 담당" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 사용자가 할 수 있는 일

사용자는 Clawket을 최신 상태로 유지하기 위해 두 가지 동작을 할 수 있다.

하나는 **최신 버전 설치**다. 명령 한 번으로 최신 릴리스를 내려받아 로컬의 CLI와 데몬 바이너리를 최신본으로 교체한다. 원한다면 특정 버전을 콕 집어 지정할 수도 있고, 실제로 설치하지 않고 "무엇이 받아질지"만 미리 확인하는 미리보기 방식으로 실행할 수도 있다.

다른 하나는 **업데이트 확인**이다. 아무것도 설치하지 않고, 지금 쓰는 버전이 최신인지 더 새 버전이 나왔는지만 가볍게 확인한다. 설치를 결정하기 전에 먼저 살펴볼 때 쓴다.

교체는 안전하게 일어난다. 새 바이너리로 바꾸더라도 이미 떠 있는 데몬 프로세스는 다음 재시작 전까지 계속 동작하므로, 업데이트 도중에 진행 중인 작업이 끊기지 않는다.

## 행위

- **설치(update)**: 최신 릴리스를 받아 CLI와 데몬 바이너리를 교체한다. 특정 버전 지정과 미리보기(설치 없이 받을 대상만 출력)를 선택할 수 있다.
- **버전 확인(version-check)**: 설치하지 않고, 현재 버전과 최신 릴리스를 비교해 더 새 버전이 있는지만 알려 준다.

## 시스템 흐름

CLI는 두 동작을 모두 로컬 데몬에 위임한다. 사용자가 설치를 요청하면 CLI는 데몬의 자체 업데이트 처리 경로로 요청을 넘기고(미리보기 여부와 지정 버전을 함께 전달), 데몬이 GitHub Releases에서 산출물을 받아 무결성을 검증한 뒤 바이너리를 원자적으로 교체한다. 업데이트 확인 역시 CLI가 데몬의 버전 확인 경로를 호출해 결과를 받아 보여 준다.

이처럼 다운로드·검증·교체라는 무거운 작업은 데몬이 책임지고, CLI는 사용자의 의도를 데몬에 전달하고 결과를 출력하는 얇은 진입점 역할을 한다. 외부 네트워크로 나가는 경로는 GitHub Releases 한 곳뿐이다.

## 어디에 구현되어 있나

업데이트와 버전 확인 명령은 CLI 바이너리에 정의되어 있으며, 각각 데몬의 자체 업데이트 엔드포인트와 버전 확인 엔드포인트를 호출하는 전달 계층으로 구현된다.

근거: `cli/src/main.rs`의 update 명령 선언(dry_run·version 옵션)과 핸들러(데몬 self-update 경로로 전달), version-check 명령 선언과 핸들러(데몬 version-check 경로 호출).

## 미확정 (OPEN)
- [ ] OPEN: 무결성 검증(SHA256 등)과 원자적 교체의 구체 동작이 데몬 측 self-update 처리에 있는지 미확인(설치 게이트의 검증 로직과 동일 경로인지 미검증).
- [ ] OPEN: 데몬 자기 자신을 교체할 때의 재시작 책임 주체(사용자 수동 재시작 vs 자동)와 그 시점 미확인.
