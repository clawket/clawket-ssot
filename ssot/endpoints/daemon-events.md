---
id: endpoint.daemon-events
kind: Endpoint
title: 데몬 GET /events (SSE)
definition: 태스크·사이클·실행 기록의 변경을 실시간으로 흘려보내는 이벤트 스트림. 웹 대시보드와 CLI watch가 이 스트림을 구독해 화면·터미널을 즉시 갱신한다.
realizedBy:
  - component.daemon
implementedIn:
  - clawket/README.md
relatesTo:
  - to: component.daemon
    type: backed-by
    note: 이벤트 버스를 제공하는 로컬 데몬
  - to: concept.task
    type: reads
    note: 태스크 변경을 이벤트로 흘려보낸다
  - to: concept.cycle
    type: reads
    note: 사이클 변경을 이벤트로 흘려보낸다
  - to: concept.run
    type: reads
    note: 실행 기록 변경을 이벤트로 흘려보낸다
governedBy: []
impacts:
  - component.web-dashboard
consumedBy:
  - component.web-dashboard
  - endpoint.cli-watch
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
작업 상태가 바뀔 때마다 그 변화를 구독자에게 즉시 밀어주는 실시간 통지 통로다. 누군가 태스크를 시작·완료하거나 사이클을 활성화하거나 실행 기록이 생기면, 화면을 새로고침하지 않아도 변경이 바로 반영되도록 데몬이 변경 사실을 흘려보낸다. 사람이 보는 웹 대시보드의 "실시간 업데이트"와 터미널에서 흐름을 지켜보는 CLI watch가 모두 이 통로를 통해 같은 변경을 본다.

## 요청 / 응답
구독을 시작할 때 관심 범위(특정 프로젝트·태스크·사이클)를 좁히기 위한 필터를 전달할 수 있다. 응답은 한 번에 끝나는 것이 아니라 연결을 열어둔 채 변경이 생길 때마다 한 건씩 이어 보내는 지속 스트림이다. 각 이벤트는 "무엇이(태스크/사이클/실행) 어떻게 바뀌었는지"를 알리는 신호이며, 구독 측은 이를 받아 해당 항목만 갱신한다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 구독할 수 있다. 연결은 한쪽이 끊을 때까지 열려 있는 단방향 푸시(서버→클라이언트)이며, 구독 측이 명시한 필터로 받는 이벤트 범위를 좁힌다. 데몬이 내려가면 스트림이 끊기고, 구독 측(대시보드·watch)은 재연결로 복구한다.

## provenance
이벤트 버스(`/events`)의 존재와 SSE 방식·구독 주체는 `clawket/README.md`(아키텍처 다이어그램의 "SSE event bus (/events)", 웹 대시보드 실시간 업데이트, `clawket watch` 설명)에 근거한다. 제공 주체는 로컬 데몬(component.daemon), 소비 주체는 웹 대시보드(component.web-dashboard)와 `clawket watch`(endpoint.cli-watch)다.

## 미확정 (OPEN)
- [ ] OPEN: 이벤트 페이로드의 구체적 의미 단위(변경 종류 표기·증분 vs 전체 갱신 여부)와 지원하는 필터 키 목록은 데몬 라우터 코드를 정독해 확인 필요.
