---
id: capability.register-project
kind: Capability
title: 프로젝트 등록
definition: 사람이나 LLM 에이전트가 하나의 작업 디렉터리를 Clawket 프로젝트로 등록해, 그 폴더에서 일어나는 모든 작업이 구조화된 상태(플랜·태스크·이력)로 추적되고 가드레일(훅 강제)이 켜지도록 만드는 일이다.
purpose: 작업 디렉터리를 프로젝트로 등록해 구조화된 작업 추적과 훅 강제를 활성화한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
  - component.sqlite-storage
implementedIn:
  - cli/src/main.rs
  - daemon/src/routes/projects.rs
relatesTo:
  - { to: concept.project, type: mutates, note: "등록은 새 프로젝트 엔티티를 생성한다" }
  - { to: endpoint.cli-project, type: calls, note: "project create / resolve 명령으로 수행" }
  - { to: domain.hook-enforcement, type: relates-to, note: "등록·enabled 상태가 훅 강제의 전제" }
impacts:
  - concept.project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

Clawket을 쓰려는 첫 단계다. 어떤 폴더에서 구조화된 작업(플랜을 세우고 태스크를 만들어 진행 이력을 남기는 일)을 시작하려면, 그 폴더가 먼저 "프로젝트"로 등록되어 있어야 한다. 등록되지 않은 디렉터리에서는 가드레일이 그 폴더를 관리 대상이 아닌 것으로 보고 아무 제약도 걸지 않는다 — 즉 구조화된 작업의 보호 장치가 작동하지 않는다.

등록의 의도는 두 가지다. 첫째, 그 폴더에서 일어나는 변경을 영구히 추적할 수 있는 상태 계층의 출발점을 만드는 것. 둘째, "활성 태스크 없이는 변경 작업을 막는다"는 가드레일이 작동할 수 있도록 그 폴더를 Clawket의 관리 범위 안에 들이는 것이다.

## 행위

- 새 이름과 작업 디렉터리(cwd)를 지정해 프로젝트를 생성한다. 하나의 프로젝트에는 하나 이상의 디렉터리가 묶일 수 있다.
- 현재 작업 디렉터리가 어느 프로젝트에 속하는지 되짚어(resolve) 확인한다 — 훅과 대시보드가 "지금 이 폴더가 어떤 프로젝트인가"를 판단하는 근거다.
- 등록 후에는 프로젝트가 웹 대시보드 사이드바에 나타나, 사람도 같은 상태를 볼 수 있다.

## 시스템 흐름

사용자(또는 에이전트)가 "이 폴더를 프로젝트로 등록해" 라고 하면 → 에이전트가 CLI로 프로젝트 생성을 호출하고 → CLI는 로컬 데몬의 HTTP API를 거쳐 → 데몬이 SQLite에 프로젝트 레코드를 만든다. 이후 세션이 시작될 때마다 훅이 현재 디렉터리를 이 등록 정보와 대조해, 등록된(그리고 활성화된) 프로젝트면 가드레일을 적용한다.

## 어디에 구현되어 있나

CLI의 명령 진입점은 `cli/src/main.rs`의 project 서브커맨드, 실제 생성·resolve 처리는 데몬 측 `daemon/src/routes/projects.rs`에 있다. 생성된 상태는 로컬 SQLite에 영구 저장된다. 등록을 끄거나 다시 켜는 동작(enabled 토글)은 별도 역량(프로젝트 훅 비활성화)에서 다룬다.

## 미확정 (OPEN)
- [ ] OPEN: 한 프로젝트에 다중 cwd를 바인딩할 때의 충돌·우선순위 규칙은 코드에서 추가 확인 필요.
