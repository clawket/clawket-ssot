---
id: capability.config-completions
kind: Capability
title: 설정 관리·셸 자동완성 생성
definition: 사용자가 Clawket의 동작 설정값을 읽고 쓰며, 자신이 쓰는 셸(bash/zsh/fish/powershell/elvish)에 맞는 명령어 자동완성 스크립트를 생성하도록 돕는 기능.
purpose: "사용자가 Clawket의 기본 동작(기본 프로젝트, 데몬 포트 등)을 자기 환경에 맞게 조정하고, 터미널에서 명령을 빠르고 오타 없이 입력할 수 있게 한다."
servesPersona: [persona.human-operator, persona.llm-coding-agent]
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.cli-config, type: calls, note: "설정 읽기/쓰기는 config 명령으로 수행" }
  - { to: endpoint.cli-completions, type: calls, note: "자동완성 스크립트 생성은 completions 명령으로 수행" }
  - { to: component.sqlite-storage, type: depends-on, note: "설정값의 실제 저장은 데몬이 담당" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 사용자가 할 수 있는 일

이 기능은 두 가지 서로 다른 편의 작업을 묶는다.

첫째, **설정 관리**다. Clawket이 어떻게 동작할지를 결정하는 설정값(예: 기본으로 사용할 프로젝트, 데몬이 들을 포트 같은 환경 설정)을 사용자가 직접 읽고, 새로 정하고, 지울 수 있다. 매번 명령마다 옵션을 길게 붙이지 않아도 자주 쓰는 기본값을 한 번 정해 두면 그 뒤로는 그대로 적용된다. 설정은 사용자 홈 디렉터리 아래의 Clawket 설정 위치(`~/.config/clawket/`)에 보관된다.

둘째, **셸 자동완성 생성**이다. 사용자가 쓰는 터미널 셸 종류를 지정하면, 그 셸에서 `clawket` 명령과 하위 명령·옵션을 탭 키로 자동완성할 수 있는 스크립트를 만들어 준다. 이 스크립트를 셸 설정에 한 번 넣어 두면, 이후로는 명령 이름을 끝까지 외우지 않아도 되고 오타로 인한 실패가 줄어든다.

## 행위

- **설정값 읽기**: 키를 지정해 현재 값을 본다(예: 기본 프로젝트, 데몬 포트). 전체 설정을 한 번에 나열할 수도 있다.
- **설정값 쓰기**: 키와 값을 지정해 새로 정하거나 덮어쓴다.
- **설정값 지우기**: 키를 지정해 해당 설정을 제거한다(기본 동작으로 되돌아간다).
- **자동완성 스크립트 생성**: 셸 종류(bash · zsh · fish · powershell · elvish 중 하나)를 지정하면 그 셸용 자동완성 스크립트를 표준 출력으로 내보낸다. 지원하지 않는 셸 이름을 주면 오류를 알리고 멈춘다.

## 시스템 흐름

설정 관리와 자동완성 생성은 처리 경로가 다르다.

- **설정 관리**는 로컬 데몬에 위임된다. 사용자가 설정을 읽거나 바꾸거나 지우면 CLI는 그 요청을 데몬의 설정 API로 전달하고, 설정값의 실제 보관과 조회는 데몬이 맡는다. 따라서 설정 명령은 데몬이 떠 있어야 동작한다.

- **자동완성 생성**은 데몬 없이 CLI 안에서 완결된다. CLI가 자기 명령 구조를 그대로 읽어 지정된 셸 형식의 스크립트를 만들어 출력할 뿐이라, 네트워크나 데몬 연결이 필요 없다. 이 때문에 자동완성 생성은 설치 직후나 데몬이 꺼진 상태에서도 쓸 수 있다.

## 어디에 구현되어 있나

두 명령 모두 CLI 바이너리에 정의되어 있다. 설정 명령은 데몬의 설정 엔드포인트를 호출하는 얇은 전달 계층이고, 자동완성 명령은 CLI 자체의 명령 정의로부터 셸별 스크립트를 만들어 내는 순수 로컬 동작이다.

근거: `cli/src/main.rs`의 completions 명령 선언과 핸들러(셸 분기: bash/zsh/fish/powershell/elvish), config 명령 선언과 핸들러(데몬 설정 API로의 get/set/unset/list 전달).

## 미확정 (OPEN)
- [ ] OPEN: 설정 키의 전체 목록과 각 키의 의미·기본값(예: default_project, daemon.port 외)이 어디에 정의되는지 미확인.
- [ ] OPEN: 설정값이 SQLite에 저장되는지 별도 설정 파일에 저장되는지 데몬 측 저장 위치 미확인(설명에는 `~/.config/clawket/`로 표기되나 데몬 위임 경로와의 정합 미검증).
