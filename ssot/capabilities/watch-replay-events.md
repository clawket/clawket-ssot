---
id: capability.watch-replay-events
kind: Capability
title: 실시간 관찰·실행 리플레이·감사 이력 재생
purpose: "작업 중인 변화를 실시간으로 지켜보고, 끝난 태스크의 실행 이력을 되짚어 보며, 과거 감사 기록을 다시 흘려보내 무슨 일이 언제 일어났는지 사후에 확인한다."
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.cli-watch, type: realizes, note: "watch 명령으로 데몬의 라이브 엔티티 변경 이벤트를 SSE로 실시간 스트리밍한다." }
  - { to: endpoint.cli-replay, type: realizes, note: "replay 명령으로 한 태스크의 실행(run) 이력을 순서대로 재생한다." }
  - { to: endpoint.daemon-events, type: depends-on, note: "라이브 스트림과 감사 이력 재생 모두 데몬의 이벤트 스트림(SSE)에 의존한다." }
  - { to: concept.run, type: relates-to, note: "리플레이가 재생하는 단위는 태스크별 실행 기록(run)이다." }
  - { to: concept.audit-log, type: depends-on, note: "감사 이력 재생은 변조 탐지 가능한 감사 로그 항목을 유한 SSE 스트림으로 다시 흘려보낸다." }
impacts:
  - concept.run
  - concept.audit-log
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

작업 상태가 어떻게 변하고 있는지를 세 가지 시간축으로 들여다본다. 현재(라이브)에는 태스크·플랜·유닛·사이클·지식·실행이 바뀌는 순간을 실시간 스트림으로 받아 본다. 과거(사후 점검)에는 특정 태스크가 어떤 실행들을 거쳤는지 순서대로 되짚어 본다. 더 거슬러 올라가서는, 누가·언제·무엇을 바꿨는지 남아 있는 감사 기록을 다시 한 번 흘려보내 검토한다.

이를 통해 운영자는 진행 상황을 모니터링하거나 문제를 사후 분석할 수 있고, LLM 에이전트는 자기 작업의 흔적과 맥락을 시간 순으로 복원할 수 있다.

## 행위

- 라이브 관찰: 데몬에서 발생하는 엔티티 변경 이벤트를 Server-Sent-Events로 실시간 받아 출력한다. 프로젝트·태스크·사이클로 필터링하고 텍스트 또는 JSON 한 줄 형식으로 받는다. 중단할 때까지 계속 흐른다.
- 실행 리플레이: 한 태스크의 실행 기록들을 순서대로 재생해 각 실행의 시작·종료, 담당 에이전트, 결과, 노트를 보여준다. 재생할 실행 개수를 제한할 수 있다.
- 감사 이력 재생: 과거 감사 로그 항목들을 유한한 SSE 스트림으로 다시 흘려보낸다. 데몬이 한 항목씩 이벤트로 내보낸 뒤 스트림을 닫고, 엔티티 종류·엔티티 ID로 범위를 좁히거나 최대 개수를 정할 수 있다.

## 시스템 흐름

라이브 관찰과 감사 이력 재생은 모두 데몬의 이벤트 스트림에 붙는다. 라이브 관찰은 무기한 열린 채로 새 변경이 생길 때마다 한 줄씩 받고, 감사 이력 재생은 데몬이 저장된 감사 로그를 한 항목씩 이벤트로 내보낸 뒤 스트림을 닫는 유한 재생이다. 실행 리플레이는 데몬에 보관된 해당 태스크의 실행 기록을 조회해 시간 순으로 출력한다. 세 경로 모두 데몬이 단일 진실 저장소에서 읽어 내보내며 CLI는 그 흐름을 렌더링한다.

## 어디에 구현되어 있나

CLI(Rust 바이너리)에 watch(라이브 스트림), replay(태스크 실행 이력 재생), events replay(감사 로그 유한 재생) 명령이 정의되어 있다. 이벤트 발생·저장·스트리밍의 실체는 데몬 측에 있고 CLI는 구독·필터·출력 형식만 담당한다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 이벤트 버스 내부 구현과 감사 로그 재생 시 적용되는 기본 한도·정렬 보장의 세부는 데몬 코드 확인 필요.
