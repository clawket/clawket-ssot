---
id: endpoint.init
kind: Endpoint
title: clawket init
definition: "신규 사용자가 약 5분 안에 첫 태스크를 닫도록 프로젝트+승인된 플랜+유닛+활성 사이클+첫 진행 중 태스크를 한 번에 만들어 주는 멱등 온보딩 명령(--tutorial)."
realizedBy: [component.cli, component.daemon-client]
implementedIn: [cli/src/main.rs, cli/src/init.rs]
relatesTo:
  - { to: concept.project, type: mutates, note: "온보딩 프로젝트를 생성하거나 기존 cwd 프로젝트를 재사용" }
  - { to: concept.plan, type: mutates, note: "튜토리얼 플랜을 만들어 곧바로 승인(active)" }
  - { to: concept.unit, type: mutates, note: "플랜 아래 그룹핑 유닛 생성" }
  - { to: concept.cycle, type: mutates, note: "유닛에 묶인 사이클을 만들어 활성화" }
  - { to: concept.task, type: mutates, note: "첫 태스크를 생성해 진행 중으로 시작" }
governedBy: []
impacts: []
consumedBy: [capability.onboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
Clawket를 처음 쓰는 사람이 워크플로우 계층(프로젝트→플랜→유닛→사이클→태스크)을 손으로 하나씩 만들지 않고도, 약 5분 안에 "첫 태스크를 시작해서 닫아 보는" 경험까지 가도록 발판을 한 번에 깔아 주는 온보딩 명령이다. 튜토리얼 모드로 실행하면 작업 디렉터리를 프로젝트로 등록하고, 그 안에 "Hello Clawket" 플랜을 만들어 곧바로 승인(시작 가능 상태)으로 올리고, 그 아래 유닛과 활성 사이클을 만든 뒤, 첫 태스크를 진행 중 상태로 시작시켜 준다.

핵심 성질은 멱등성이다. 이미 한 번 발판을 깐 작업 디렉터리에서 다시 실행하면 똑같은 것을 또 만들어 중복으로 쌓지 않고, 이미 있는 프로젝트를 재사용하면서 기존 플랜 목록과 "상태를 보려면 대시보드를 보라"는 안내만 출력하고 끝낸다.

## 요청 / 응답
- 보낼 수 있는 것: 튜토리얼 모드 켜기(현재는 이 모드만 지원), 발판을 깔 작업 디렉터리(기본은 현재 디렉터리이며, 없으면 만들어 준다).
- 돌려주는 것: 생성·재사용 진행 상황. 새로 깔 때는 만들어진 프로젝트·플랜·유닛·사이클·태스크가 단계별로 표시되고, 이미 발판이 있는 경우에는 "이미 깔려 있어 건너뛴다"는 안내와 기존 플랜 목록, 그리고 상태 확인용 다음 명령 힌트가 나온다.

## 권한 / 제약
로컬에서 그 사용자 계정으로 실행하며 데몬이 떠 있어야 한다. 튜토리얼 모드 없이 실행하면 "현재는 튜토리얼만 지원한다"며 모드를 켜라고 안내하고 멈춘다. 지정한 작업 디렉터리가 없으면 만들어 주고, 실제 경로로 정규화한 뒤 등록한다. 재실행 시 중복 생성을 막기 위해, 해당 프로젝트에 이미 플랜이 있으면 새로 만들지 않고 기존 발판을 보여 주는 쪽으로 단락(short-circuit)한다.

## provenance
명령 정의와 옵션(튜토리얼 모드·작업 디렉터리)은 CLI 진입점(`cli/src/main.rs`)에, 프로젝트 보장→플랜 생성·승인→유닛→사이클→첫 태스크로 이어지는 발판 흐름과 멱등 단락 처리는 `cli/src/init.rs`에 있다. 제공 주체는 CLI 바이너리이며 각 엔티티 생성은 데몬 HTTP 호출(데몬 클라이언트 컴포넌트)을 통한다.

## 미확정 (OPEN)
- [ ] OPEN: 튜토리얼 외 비-튜토리얼 init이 향후 어떤 동작을 가질지(현재는 미지원) 로드맵 확인 필요.
