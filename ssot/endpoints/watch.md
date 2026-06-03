---
id: endpoint.watch
kind: Endpoint
title: clawket watch
definition: "데몬의 실시간 변경 이벤트 스트림을 구독해 엔티티 변화를 발생하는 즉시 보여주는 CLI 명령(Ctrl-C 까지 무한 실행)."
realizedBy:
  - component.cli
  - component.daemon-client
implementedIn:
  - cli/src/main.rs
  - cli/src/client.rs
relatesTo:
  - to: endpoint.daemon-events
    type: calls
    note: "데몬의 라이브 이벤트 스트림(/events)을 구독한다."
  - to: concept.task
    type: reads
    note: "태스크 변경 이벤트를 수신한다."
  - to: concept.cycle
    type: reads
    note: "사이클 변경 이벤트를 수신한다."
  - to: concept.run
    type: reads
    note: "런 변경 이벤트를 수신한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

`clawket watch` 는 데몬이 발생시키는 작업 상태 변화를 사람이나 에이전트가 실시간으로 따라볼 수 있게 해 주는 관찰용 명령이다. 한 번 실행하면 끝나지 않고, 새 이벤트가 들어올 때마다 한 줄씩 출력하면서 사용자가 Ctrl-C 로 중단할 때까지 계속 떠 있는다. 태스크가 시작·완료되거나, 사이클이 활성화되거나, 런이 기록되는 등 워크스페이스 안에서 무슨 일이 벌어지는지를 별도의 폴링 없이 즉시 파악하려는 상황에서 쓴다.

스트리밍은 데몬이 제공하는 라이브 이벤트 채널을 구독하는 방식이다. 일반 조회 명령과 달리 응답 전체를 모아서 한 번에 보여 주는 것이 아니라, 도착하는 이벤트마다 화면에 흘려 보내므로 파이프로 연결된 후속 처리도 이벤트를 받는 즉시 처리할 수 있다.

## 요청 / 응답

요청 시 관심 범위를 세 가지 필터로 좁힐 수 있다 — 특정 프로젝트, 특정 태스크, 특정 사이클. 아무 필터도 주지 않으면 데몬이 내보내는 모든 변경 이벤트를 받는다. 출력 형식은 두 가지를 고를 수 있다: 사람이 읽기 좋은 기본 텍스트 형식(이벤트 식별자·이벤트 이름·내용을 한 줄로)과, 후속 도구가 파싱하기 좋은 줄 단위 JSON 형식(이벤트 식별자·이름·내용을 구조화된 객체로). 각 이벤트는 어떤 종류의 변화인지(태스크/플랜/유닛/사이클/지식/런)와 그 변화의 내용을 담는다.

## 권한 / 제약

데몬이 실행 중이어야 한다 — 데몬에 연결할 수 없으면 "데몬이 떠 있는지 확인하라"는 안내와 함께 실패한다. 이 명령은 읽기 전용 관찰이며 어떤 상태도 바꾸지 않는다. 무한 스트림이므로 자동화 스크립트에서 쓸 때는 종료 조건을 호출 측에서 책임져야 한다.

## provenance

CLI 진입점 정의와 출력 처리는 `cli/src/main.rs`(Watch 서브커맨드 정의 및 디스패치), SSE 라인 단위 파싱·증분 출력은 `cli/src/client.rs`(stream_sse)에 있다. 실제 이벤트 공급원은 데몬의 라이브 이벤트 스트림 엔드포인트다. 코드가 사라져도, "데몬의 실시간 변경 이벤트를 프로젝트/태스크/사이클 단위로 걸러 텍스트 또는 JSON 으로 흘려 보내는, 중단 전까지 무한 실행되는 관찰 명령"이라는 의미만으로 재구현이 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 라이브 이벤트 채널이 내보내는 이벤트 종류·이름 카탈로그의 정확한 목록과 페이로드 의미를 데몬 라우터에서 확인할 것.
