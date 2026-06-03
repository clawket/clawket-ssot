---
id: endpoint.replay
kind: Endpoint
title: clawket replay
definition: "한 태스크의 실행 이력(런)을 시작·종료·에이전트·결과·노트까지 순서대로 재생해 사후 점검하는 CLI 명령."
realizedBy:
  - component.cli
  - component.daemon-client
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: endpoint.daemon-runs
    type: reads
    note: "데몬에서 해당 태스크의 런 이력을 조회한다(/runs/replay)."
  - to: concept.run
    type: lists
    note: "태스크에 속한 런 기록을 시간순으로 나열한다."
  - to: concept.task
    type: reads
    note: "지정한 태스크를 대상으로 동작한다."
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

`clawket replay` 는 특정 태스크가 그동안 어떻게 실행되어 왔는지를 되짚어 보는 사후 점검(post-mortem) 명령이다. 태스크 하나에는 여러 번의 실행 기록(런)이 쌓일 수 있는데, 이 명령은 그 런들을 순서대로 펼쳐 보여 준다. 무엇이 언제 시작·종료됐는지, 어떤 에이전트가 돌렸는지, 결과가 무엇이었고 어떤 노트가 남았는지를 한 흐름으로 읽어, 실패 원인 분석이나 진행 경과 추적에 쓴다.

라이브 스트림인 watch 와 달리 이쪽은 이미 끝난 기록을 되감아 보는 것이라, 한 번 출력하고 종료한다.

## 요청 / 응답

요청에는 재생할 대상 태스크를 지정한다. 추가로 재생할 런의 개수를 제한할 수 있으며 기본값은 10개다(최근 것 위주). 응답은 해당 태스크의 런 목록으로, 각 런마다 시작·종료 시각, 실행 주체(에이전트), 결과, 남긴 노트가 시간 순서대로 담긴다.

## 권한 / 제약

데몬이 실행 중이어야 한다. 읽기 전용이며 상태를 바꾸지 않는다. 런 개수 제한 기본값(10) 때문에 더 오래된 이력까지 보려면 제한값을 키워야 한다.

## provenance

CLI 정의·디스패치는 `cli/src/main.rs`(Replay 서브커맨드: task 인자와 limit 옵션을 받아 데몬의 런 재생 조회를 호출). 데이터 출처는 데몬의 런 이력 조회 경로다. "한 태스크의 런 기록을 개수 제한과 함께 시간순으로 되감아 보여 주는 사후 점검 명령"이라는 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 런 재생 응답이 각 런별로 정확히 어떤 필드를 어떤 의미로 돌려주는지(에이전트/결과/노트의 구조)를 데몬 라우터에서 확인할 것.
