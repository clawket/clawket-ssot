---
id: capability.onboard
kind: Capability
title: 5분 온보딩 스캐폴딩
purpose: "신규 사용자가 빈 상태에서 헤매지 않도록, 프로젝트부터 첫 진행 중 태스크까지 한 번에 만들어 주어 약 5분 내 첫 태스크를 닫을 수 있게 한다."
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
  - cli/src/init.rs
relatesTo:
  - { to: endpoint.cli-init, type: realizes, note: "init --tutorial 명령으로 온보딩 스캐폴드를 실행한다." }
  - { to: concept.project, type: relates-to, note: "스캐폴딩의 첫 단계로 작업 디렉터리를 프로젝트로 등록한다(이미 등록된 cwd는 재사용)." }
  - { to: concept.plan, type: relates-to, note: "승인된(active) 플랜을 만들어 태스크 시작 자격을 곧바로 갖춘 상태로 둔다." }
  - { to: concept.cycle, type: relates-to, note: "활성 사이클을 만들어 첫 태스크가 시작 가능한 상태가 되게 한다." }
  - { to: concept.task, type: relates-to, note: "스캐폴딩의 종착점으로 첫 태스크를 진행 중 상태로 만든다." }
impacts:
  - concept.project
  - concept.plan
  - concept.cycle
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

처음 Clawket을 쓰는 사람이 워크플로우 계층(프로젝트→플랜→유닛→사이클→태스크)을 하나씩 손으로 세우지 않아도, 단 한 번의 명령으로 전체 골격이 갖춰진다. 작업 디렉터리가 프로젝트로 등록되고, 승인된 플랜과 그 안의 유닛, 활성 사이클, 그리고 이미 진행 중 상태인 첫 태스크까지 한 번에 만들어진다. 즉, 사용자는 곧바로 "닫을 수 있는 태스크"를 손에 쥐고 시작하며, 빈 상태에서 무엇부터 해야 하는지 몰라 막히는 일이 없다.

같은 작업 디렉터리에서 다시 실행해도 안전하다 — 이미 등록된 프로젝트는 새로 만들지 않고 재사용하므로, 중복 골격이 쌓이지 않는다.

## 행위

- 온보딩 스캐폴드 실행: 튜토리얼 모드로 프로젝트·승인된 플랜·유닛·활성 사이클·진행 중인 첫 태스크를 한 번에 생성한다.
- 작업 디렉터리 지정: 등록할 디렉터리를 직접 지정하거나 현재 디렉터리를 사용한다. 지정한 경로가 없으면 만들어 준다.
- 멱등 재실행: 이미 플랜이 있는 등록된 프로젝트에서 다시 실행하면 기존 프로젝트를 재사용해 중복을 피한다.

## 시스템 흐름

명령을 실행하면 먼저 대상 작업 디렉터리를 정규화하고 없으면 생성한다. 이후 데몬에 프로젝트를 보장(없으면 등록, 있으면 재사용)하고, 그 위에 승인된 플랜, 유닛, 활성 사이클을 차례로 만든 뒤 마지막으로 첫 태스크를 진행 중 상태로 둔다. 각 단계는 데몬의 생성 경로를 통해 이뤄지며, 결과적으로 워크플로우 게이트(플랜이 active여야 태스크 시작 가능, 태스크는 활성 사이클에 묶여야 시작 가능)를 이미 통과한 상태가 만들어진다.

## 어디에 구현되어 있나

CLI(Rust 바이너리)에 init 명령이 정의되어 있고, 6단계 스캐폴딩(프로젝트 보장 → 플랜 → 유닛 → 사이클 → 첫 태스크)의 실제 로직은 init 모듈에 있다. 각 엔티티 생성은 데몬을 통해 수행된다.

## 미확정 (OPEN)
- [ ] OPEN: 스캐폴딩 6단계의 정확한 순서와 각 단계가 만드는 기본 제목·내용(튜토리얼 콘텐츠) 세부는 init 모듈 전체 확인 필요.
