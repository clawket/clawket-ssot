---
id: integration.claude-code-adapter
kind: Integration
title: 데스크탑에서 본 Claude Code 어댑터(실행 기록의 외부 생산자)
purpose: 작업의 실행 기록(Run)은 데스크탑이 만드는 것이 아니라, Claude Code 어댑터가 작업을 시작하고 끝낼 때 자동으로 만들고 닫는다. 데스크탑은 그 기록을 읽어 보여 주기만 하는 소비자다. 이 역할 분담을 분명히 해 두어, 데스크탑이 실행 기록을 임의로 만들거나 고치지 않도록 한다.
definition: 작업 시작·종료 시점에 실행 기록(Run)을 생성·종료하는 외부 주체(Claude Code 어댑터)와, 그 기록을 읽기 전용으로만 표시하는 데스크탑 사이의 연동이다. 데스크탑은 실행 기록의 생산자가 아니라 소비자다.
integratesWith:
  - component.desktop-app
  - component.daemon-client
  - component.claude-adapter
consumesApi:
  - endpoint.daemon-runs
impacts:
  - concept.run
relatesTo:
  - to: concept.run
    type: relates-to
    note: 이 연동이 다루는 데이터는 작업별 실행 기록이다.
  - to: integration.claude-code-hooks
    type: relates-to
    note: 실행 기록을 만들고 닫는 어댑터의 동작은 Claude Code 훅 이벤트에서 트리거된다.
  - to: endpoint.daemon-runs
    type: depends-on
    note: 데스크탑은 데몬의 실행 기록 목록 API를 읽기 전용으로 호출한다.
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

작업 하나에는 그 작업이 실제로 돌아간 이력인 실행 기록(Run)이 남는다. 이 기록을 누가 만드느냐가 이 연동의 핵심이다 — 데스크탑 앱이 아니라, Claude Code 어댑터가 만든다. 어댑터는 작업이 시작될 때 실행 기록을 새로 만들고, 작업이 끝날 때 그 기록을 닫는다(시작·종료 시각, 어떤 에이전트가 돌았는지, 결과·노트 등을 채운다).

데스크탑은 이 흐름에서 철저히 읽기 쪽이다. 작업 상세 화면에서 실행 이력을 보여 주기 위해 데몬에서 실행 기록 목록을 가져와 표시할 뿐, 실행 기록을 만들거나 끝내는 쓰기 동작은 하지 않는다. 화면 쪽 HTTP 클라이언트의 실행 기록 관련 기능에는 "목록 조회"만 있고 생성·종료 기능이 없으며, 코드 주석에도 "실행 기록은 Claude Code 어댑터가 작업 시작/종료 시 만들고 닫는다 — 데스크탑에서는 읽기 전용"이라고 명시되어 있다.

이렇게 역할을 나눈 이유는, 실행 기록이 "실제로 누가 언제 작업을 돌렸는가"라는 사실의 기록이기 때문이다. 사람이 보는 화면에서 이를 임의로 만들거나 수정하면 기록의 신뢰성이 깨진다. 그래서 생산은 작업을 실제로 수행하는 주체(어댑터)에게만 맡기고, 데스크탑은 보여 주는 역할로 한정한다.

## 구현 위치 (provenance)

- `desktop/apps/desktop/src/data/api.ts` — 화면 쪽 HTTP 클라이언트. 실행 기록 영역에는 작업/프로젝트로 거른 "목록 조회"만 있고 생성·종료 메서드가 없다. 코드 주석이 읽기 전용 설계와 그 이유(어댑터가 생산자)를 못 박아 둔다.
- `desktop/apps/desktop/src/data/types.ts` — 데스크탑이 다루는 실행 기록의 데이터 형태(읽어서 표시할 필드들) 정의.

이 연동을 사용하는 내부 컴포넌트는 데스크탑 앱(component.desktop-app)과 그 HTTP 클라이언트(component.daemon-client)이며, 실행 기록을 실제로 생산하는 외부 주체는 Claude 어댑터(component.claude-adapter)다. 데스크탑과 어댑터는 직접 통신하지 않고, 둘 다 데몬을 통해 같은 실행 기록 데이터를 공유한다 — 어댑터는 쓰고, 데스크탑은 읽는다.

## 불변식

- 데스크탑은 실행 기록에 대해 읽기 전용이다. 생성·종료는 작업을 실제로 수행하는 어댑터의 책임이며, 사람이 보는 화면에서 실행 기록을 임의로 만들거나 닫지 않는다.

## 영향 범위

- 이 연동이 다루는 핵심 데이터는 실행 기록(concept.run)이다. 어댑터가 기록을 만들고 닫는 시점·내용이 곧 데스크탑이 보여 줄 이력의 내용이 된다.

## 미확정 (OPEN)
- [ ] OPEN: 읽기 전용 원칙을 강제하는 불변식 노드가 별도로 존재하는지(invariant.* 부재) — 현재는 코드 주석과 클라이언트에 쓰기 메서드 부재로만 보장되며, 명시적 invariant 등록 여부 확인 필요.
- [ ] OPEN: 데스크탑이 실행 이력을 실시간으로 갱신하는지(데몬 SSE 이벤트 구독) 여부 — 본 연동 범위에서는 목록 조회만 확인됨.
