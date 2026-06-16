---
id: capability.semantic-embed
kind: Capability
title: 텍스트 임베딩·임베딩 백필
definition: 임의의 텍스트를 의미 벡터(임베딩)로 변환해 주고, 데몬이 시작될 때 아직 임베딩이 없는 과거 태스크를 찾아 자동으로 채워 넣어 의미 검색의 빈틈을 메우는 기능이다.
purpose: 태스크·지식의 의미 검색이 누락 없이 작동하도록, 텍스트를 벡터로 바꾸는 단일 창구를 제공하고 과거 데이터의 임베딩 공백을 부팅 시 메운다.
servesPersona:
  - persona.llm-coding-agent
realizedBy:
  - component.daemon
  - component.embedding-engine
  - component.embeddings
implementedIn:
  - daemon/src/routes/embed.rs
  - daemon/src/main.rs
relatesTo:
  - to: component.embedding-engine
    type: depends-on
    note: 실제 벡터 변환은 온디바이스 임베딩 엔진이 수행한다.
  - to: component.sqlite-storage
    type: relates-to
    note: 백필된 태스크 임베딩은 로컬 벡터 저장소에 적재된다.
  - to: concept.task
    type: relates-to
    note: 부팅 백필 대상은 임베딩이 비어 있는 태스크다.
impacts:
  - domain.knowledge-rag
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

LLM 코딩 에이전트(또는 다른 내부 호출자)는 짧은 문장이든 긴 본문이든 임의의 텍스트를 건네고, 그에 대응하는 의미 벡터를 돌려받을 수 있다. 이 벡터는 "글자가 똑같은가"가 아니라 "뜻이 가까운가"를 비교하는 데 쓰이며, 태스크·지식 검색에서 의미 기반 매칭의 기초가 된다.

또 하나의 가치는 자동 보정이다. 데몬이 켜질 때, 과거에 만들어졌지만 아직 의미 벡터가 만들어지지 않은 태스크들을 스스로 찾아내 한 건씩 임베딩을 채운다. 사용자가 손대지 않아도 의미 검색의 사각지대가 시간이 지나며 메워진다.

## 행위

- 변환(읽기·계산): 전달받은 텍스트를 의미 벡터로 환산해 차원 수와 벡터 값을 돌려준다. 빈 텍스트는 잘못된 요청으로 거절된다.
- 백필(자동 보정): 데몬 시작 직후, 임베딩이 비어 있는 태스크 목록을 훑어 각 태스크의 원본 텍스트를 임베딩하고 벡터 저장소에 적재한다. 적재에 실패하거나 원본이 비어 있는 항목은 건너뛰고, 완료·실패 건수를 로그로 남긴다.

## 시스템 흐름

호출자가 텍스트를 데몬의 임베딩 경로로 보내면, 데몬은 온디바이스 임베딩 엔진에 변환을 위임하고 결과 벡터를 응답으로 돌려준다. 외부 네트워크로 텍스트가 나가지 않는다.

백필은 다른 결로 동작한다. 데몬이 기동할 때 별도 비동기 작업이 한 번 떠서, 임베딩이 누락된 태스크를 골라 같은 임베딩 엔진으로 벡터를 만들고 로컬 벡터 저장소에 저장한다. 이 작업은 부팅 시 일회성으로 돌고, 처리 결과 요약을 로그에 적은 뒤 끝난다.

## 어디에 구현되어 있나

- `daemon/src/routes/embed.rs`: 텍스트 → 벡터 변환을 받는 데몬 경로. 빈 입력 거절과 차원·벡터 응답을 담당한다.
- `daemon/src/main.rs`: 데몬 부팅 시 누락 임베딩을 채우는 백필 작업을 띄우는 자리. 임베딩 엔진으로 벡터를 만들고 저장소에 적재하며 결과를 로그로 남긴다.

## 미확정 (OPEN)
- [ ] OPEN: 백필이 지식 항목(knowledge)까지 대상으로 하는지, 태스크 한정인지 — 현재 추적된 코드는 태스크 백필만 보여준다.
