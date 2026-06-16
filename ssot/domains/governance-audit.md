---
id: domain.governance-audit
kind: Domain
title: 거버넌스·감사
purpose: "누가·언제·무엇을 어떻게 바꿨는지를 로컬에 변조 탐지 가능한 형태로 빠짐없이 남겨, 에이전트와 사람이 같은 상태를 다루면서도 모든 변경에 책임 추적이 가능하게 한다. 어떤 추적 데이터도 기기 밖으로 나가지 않는다."
definition: "엔티티(프로젝트·플랜·태스크·지식 등) 변경을 행위자·작업 종류·이전값/이후값과 함께 로컬 감사 로그에 기록하고, 직전 행에서 파생한 해시 체인으로 변조 여부를 검출 가능하게 하는 책임 영역. 기록은 작성 후 수정·삭제가 막혀 있는 추가 전용(append-only)이다. 원격 텔레메트리는 없으며, 실시간 이벤트 스트림(watch)과 실행 이력 재생(replay)으로 이 추적 기록을 사람이 관찰·되짚어볼 수 있게 한다. 변경을 '강제·차단'하는 가드레일은 별도 도메인의 책임이고, 본 도메인은 일어난 변경을 '기록·관찰'하는 데 한정한다."
servesPersona: [persona.human-operator, persona.llm-coding-agent]
relatesTo:
  - { to: domain.work-management, type: depends-on, note: "감사 로그가 추적하는 변경 대상(엔티티와 그 상태)은 작업 관리 도메인이 정의한다" }
  - { to: domain.hook-enforcement, type: complements, note: "가드레일은 변경을 사전 차단하고, 본 도메인은 허용된 변경을 사후 기록한다 — 예방과 추적의 짝" }
  - { to: platform.local-first, type: relates-to, note: "추적 데이터는 로컬에만 저장되고 외부로 전송되지 않는다(no remote telemetry)" }
governedBy: []
realizedBy: [component.daemon, component.sqlite-storage]
impacts: [concept.audit-log, capability.watch-replay, endpoint.cli-watch, endpoint.cli-replay]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:domain"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드 옮겨적기 금지. -->

## 목적

에이전트와 사람이 동일한 작업 상태를 함께 다루는 환경에서는 "누가 이걸 바꿨고 왜 이렇게 됐나"가 흐려지기 쉽다. 이 도메인은 모든 엔티티 변경을 빠짐없이, 그리고 나중에 손댄 흔적을 알아챌 수 있는 형태로 로컬에 남겨 책임 추적을 보장한다. 동시에 클라우드 분석 서비스로 사용 기록을 보내는 흔한 텔레메트리와는 정반대로, **어떤 추적 데이터도 기기 밖으로 내보내지 않는다**는 것을 원칙으로 한다.

## 경계와 핵심 개념

핵심 개념은 감사 로그(`concept.audit-log`)다. 변경이 일어날 때마다 한 행이 기록되며 각 행은 다음을 담는다: 무엇이 바뀌었는지(엔티티 종류·식별자·필드), 어떤 변경인지(작업 종류와 이전값/이후값), 누가 했는지(행위자 — 에이전트·CLI·외부 API·시스템).

두 가지 성질이 이 도메인을 규정한다.
- **변조 탐지**: 각 행은 직전 행에서 파생한 해시값(FNV-1a 기반 체인)을 함께 가진다. 누군가 중간 행을 몰래 고치거나 지우면 이후 체인이 어긋나 변조를 알아챌 수 있다. 완전한 암호학적 보증(머클 트리)까지는 아니고, 로컬 데몬 환경에서 손댄 흔적을 감지하는 데 충분한 가벼운 체인이다.
- **추가 전용**: 기록은 작성 후 수정·삭제가 허용되지 않는다(읽기 전용 조회만 가능). 추적 기록 자체가 사후 조작되지 않도록 하는 1차 방어선이다.

관찰 수단으로 실시간 스트림(`endpoint.cli-watch`)과 실행 이력 재생(`endpoint.cli-replay`)을 제공한다(`capability.watch-replay`).

경계 밖(이 도메인의 책임이 아닌 것):
- 변경을 사전에 막거나 허용하는 가드레일은 훅 강제(`domain.hook-enforcement`)의 책임이다. 본 도메인은 이미 일어난 변경을 사후 기록·관찰한다.
- 추적되는 엔티티와 그 상태 모델 자체는 작업 관리(`domain.work-management`)가 정의한다.

## 기능

- **변경 감사 기록**: 모든 엔티티 변경을 행위자·작업 종류·이전/이후값과 함께 변조 탐지 가능한 체인으로 남긴다(`concept.audit-log`).
- **실시간 관찰·리플레이**: 진행 중 변경을 라이브로 스트리밍하고, 특정 태스크의 실행 이력을 순서대로 되짚어본다(`capability.watch-replay`).

## 시스템 흐름

엔티티가 바뀌면 데몬이 그 변경을 감사 로그 한 행으로 적는다. 이때 직전 행에서 파생한 해시를 함께 박아 체인을 잇는다. 기록은 로컬 저장소(`component.sqlite-storage`)에만 남고 외부로 전송되지 않는다. 사람·에이전트가 관찰하려 하면, 데몬이 같은 추적 기록을 라이브 이벤트 스트림으로 흘려보내거나(watch) 과거 실행을 시간순으로 재생한다(replay). 원천 데이터는 단일 로컬 저장소이고, 데몬이 기록·스트림·재생을 모두 중개한다.

## 다른 도메인과의 관계

- 작업 관리(`domain.work-management`)에 의존한다 — 감사가 추적하는 변경 대상이 작업 관리의 엔티티다.
- 훅 강제(`domain.hook-enforcement`)와 상호 보완한다 — 한쪽은 변경을 사전 차단(예방), 본 도메인은 허용된 변경을 사후 기록(추적). 둘이 짝을 이뤄 거버넌스를 완성한다.
- local-first 원칙(`platform.local-first`)을 따른다 — 모든 추적 데이터는 로컬에만 머문다.

## 미확정 (OPEN)
- [ ] OPEN: 해시 체인의 정확한 알고리즘이 자료마다 어긋난다. 제품 문서(README의 텔레메트리 절)는 "FNV-1a hash of the previous row"로 명시하나, 데몬 구현의 함수명·주석은 SHA-256을 표방하면서 실제로는 FNV 계열 혼합을 쓴다고 적혀 있어 의도와 명명이 불일치한다. 외부에 노출되는 진실은 README 기준 FNV-1a 기반 체인으로 본다. 구현 정리 필요 여부 확인 대상.
