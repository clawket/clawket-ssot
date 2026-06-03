---
id: endpoint.runs-list
kind: Endpoint
title: 데몬 GET·POST /runs (실행 기록 목록·생성)
definition: "태스크별 실행 기록을 조건에 맞게 목록 조회하고, 새 실행 기록을 시작하는 진입점으로, 한 태스크에 열려 있는 실행이 이미 있으면 새 실행 생성을 막는다."
realizedBy: [component.daemon]
implementedIn: ["daemon/src/routes/runs.rs", "daemon/src/repo/runs.rs"]
relatesTo:
  - { to: concept.run, type: mutates, note: "새 실행 기록을 만들고(POST) 기존 기록들을 조회한다(GET)" }
  - { to: concept.task, type: reads, note: "실행은 반드시 존재하는 태스크에 매여 있어야 하며 없는 태스크면 거절한다" }
  - { to: concept.execution-envelope, type: reads, note: "생성 시 태스크의 활성 실행 봉투 체인을 해석해 스냅샷으로 동결한다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-run, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크의 실행 기록(어떤 에이전트가 언제 그 태스크를 돌렸는지)을 다루는 두 가지 동작을 한 경로에 묶은 진입점이다. 목록 조회는 조건에 맞는 실행 기록들을 최근 시작 순으로 돌려주고, 생성은 한 태스크에 대해 새 실행을 시작하면서 그 시점의 실행 계약(봉투) 상태를 함께 고정한다. 보통 사람이 직접 부르기보다 작업 시작 훅이나 실행 도구가 자동으로 호출한다.

## 요청 / 응답
목록 조회는 태스크, 세션, 프로젝트 중 하나 이상으로 범위를 좁혀 요청할 수 있고, 조건에 맞는 실행 기록들을 시작 시각 내림차순으로 돌려준다. 프로젝트로 거를 때는 그 프로젝트에 속한 모든 태스크의 실행을 모아 준다.

생성 요청에는 어떤 태스크의 실행인지(필수), 세션 식별, 실행 주체(에이전트, 기본은 "main"), 그리고 시작 상태를 보낸다. 시작 상태는 두 가지다 — 곧바로 "진행 중(started)"으로 여는 경우와, 아직 태스크를 점유하지 않고 프롬프트·스냅샷만 받아 두는 "대기(pending)" 경우. 그 외 값은 잘못된 요청으로 거절한다. 클라이언트가 실행 봉투 스냅샷을 직접 넘기면 그대로 쓰고, 넘기지 않으면 데몬이 그 순간의 활성 봉투 체인을 해석해 동결한다. 활성 봉투가 없는 태스크면 스냅샷은 비어 둔다. 응답은 만들어진 실행 기록 한 건이며, 생성 시 실시간 이벤트("실행 생성됨")가 함께 발행된다.

## 권한 / 제약
- 생성하려는 태스크가 실제로 존재해야 한다. 없으면 찾을 수 없음으로 거절한다.
- 시작 상태는 "대기" 또는 "진행 중"만 허용한다.
- 같은 태스크에 아직 끝나지 않은(열려 있는) 실행이 이미 있으면 새 실행 생성을 거부한다. 이 경우 "이미 열린 실행이 있으니 먼저 끝내라"는 충돌 응답을 돌려주어, 한 태스크에 동시에 두 개의 미종료 실행이 생기는 것을 방지한다.

## provenance
데몬 실행 라우트(`daemon/src/routes/runs.rs`)의 목록·생성 핸들러가 진입점이고, 실제 저장·조회·미종료 실행 검사는 실행 저장소 계층(`daemon/src/repo/runs.rs`)이 담당한다. 봉투 동결은 태스크 봉투 체인 해석 로직을 거친다. 제공 주체는 로컬 데몬이며, CLI의 실행 명령과 웹 대시보드가 이 경로를 소비한다.

## 미확정 (OPEN)
- 없음.
