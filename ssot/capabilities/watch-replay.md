---
id: capability.watch-replay
kind: Capability
title: 실시간 관찰·리플레이
definition: 사람이나 LLM 에이전트가 데몬이 흘려보내는 엔티티 변경 이벤트(태스크·플랜·유닛·사이클·지식·실행)를 실시간으로 받아 흐름을 지켜보고, 특정 태스크의 과거 실행 기록을 순서대로 다시 재생해 사후에 점검하는 일이다.
purpose: 진행 중인 작업의 변화를 실시간으로 관찰하고, 끝난 태스크의 실행 이력을 시간순으로 리플레이해 무슨 일이 있었는지 사후 점검한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
relatesTo:
  - { to: endpoint.cli-watch, type: calls, note: "watch 로 task/cycle/run 등 변경을 SSE 로 실시간 스트리밍하고 project/task/cycle 로 필터한다" }
  - { to: endpoint.cli-replay, type: calls, note: "replay 로 한 태스크의 run 이력을 시간순으로 재생한다" }
  - { to: concept.run, type: reads, note: "리플레이는 태스크별 실행 기록(시작/종료·에이전트·결과·노트)을 순서대로 읽는다" }
  - { to: concept.task, type: reads, note: "관찰·리플레이의 단위는 태스크와 그 실행이다" }
impacts:
  - concept.run
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

여러 에이전트가 동시에 작업하는 상황에서 "지금 무슨 일이 벌어지고 있나"를 명령줄에서 실시간으로 지켜보는 일과, "그 태스크는 어떻게 진행됐었나"를 끝난 뒤에 되짚어 보는 일이다. 관찰(watch)은 데몬이 푸시하는 변경 이벤트 스트림을 그대로 받아, 태스크·사이클·실행 등이 바뀔 때마다 한 줄씩 흘려준다 — 특정 프로젝트·태스크·사이클로 범위를 좁혀 볼 수 있고, 사람이 읽기 좋은 텍스트 또는 다른 도구로 흘려보내기 좋은 JSON 형식 중 하나로 출력한다.

리플레이(replay)는 한 태스크의 실행 기록을 시간순으로 다시 재생해, 각 실행의 시작·종료, 어떤 에이전트가 돌았는지, 결과와 노트가 무엇이었는지를 차례대로 보여준다. 사후 점검(post-mortem) 용도다.

## 행위

- 데몬의 변경 이벤트를 실시간 스트리밍한다(watch) — 끊지 않으면 Ctrl-C 전까지 계속 흘린다.
- 관찰 범위를 프로젝트·태스크·사이클로 필터한다.
- 출력 형식을 텍스트 또는 JSON으로 고른다(이벤트마다 즉시 flush되어 파이프 하류가 곧장 받는다).
- 한 태스크의 run 이력을 시간순으로 재생한다(replay) — 재생 개수를 제한할 수 있다.

## 시스템 흐름

관찰은 데몬이 내부 이벤트 버스에서 발생시키는 엔티티 변경을 Server-Sent Events로 푸시하고, CLI(또는 웹 대시보드)가 이를 구독해 받는 구조다. 같은 이벤트 스트림을 사람의 화면과 에이전트의 명령줄이 함께 구독하므로 양쪽이 보는 변화가 어긋나지 않는다. 리플레이는 실시간 스트림이 아니라, 데몬에 저장된 특정 태스크의 과거 실행 기록을 시간순으로 조회해 재구성하는 읽기 동작이다.

## 어디에 구현되어 있나

watch·replay 명령 정의는 CLI에 있으며(필터·출력 형식·재생 개수 옵션 포함), 실시간 이벤트의 원천은 데몬의 SSE 이벤트 스트림이고 리플레이의 원천은 데몬에 보존된 태스크별 실행 기록이다. 정확한 CLI 소스 경로와 데몬 이벤트 엔드포인트는 각 endpoint 노드(endpoint.cli-watch / endpoint.cli-replay)에서 다룬다.

## 미확정 (OPEN)
- [ ] OPEN: 실시간 watch 가 구독하는 데몬 SSE 엔드포인트와, 리플레이가 읽는 실행 이력 조회 경로의 정확한 분리(라이브 스트림 vs. 유한 replay)는 데몬 라우트에서 교차 확인 필요.
