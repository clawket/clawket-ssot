---
id: endpoint.cli-dashboard
kind: Endpoint
title: clawket dashboard
definition: "활성 프로젝트의 작업 요약(진행 중인 플랜·유닛·사이클·태스크)을 한눈에 보여주는 CLI 명령으로, 세션 시작 시 에이전트의 작업 컨텍스트를 채우는 데도 쓰인다."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.project
    type: reads
    note: 작업 디렉터리로부터 어느 프로젝트의 요약을 보여줄지 결정한다
  - to: concept.plan
    type: reads
  - to: concept.cycle
    type: reads
  - to: concept.task
    type: reads
  - to: endpoint.hook-session-start
    type: relates-to
    note: 세션 시작 훅이 이 명령의 출력을 컨텍스트 시드로 사용한다
governedBy:
  - invariant.one-active-plan-per-project
impacts: []
consumedBy:
  - endpoint.hook-session-start
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

지금 이 작업 디렉터리가 어떤 프로젝트에 속해 있고, 그 안에서 무엇이 진행 중인지를 한 화면으로 요약해 주는 명령이다. 사람이 "내가 지금 어디에 있지?"라고 물을 때, 그리고 코딩 에이전트가 새 세션을 열 때 가장 먼저 보는 출력이다. 활성 플랜, 그 아래 유닛과 사이클, 그리고 현재 진행 중인 태스크를 묶어서 보여 주므로, 별도로 여러 목록 명령을 돌리지 않아도 현재 작업 상황을 파악할 수 있다.

호출하는 쪽은 둘이다. 하나는 사람이 직접 터미널에서 상황을 확인하는 경우, 다른 하나는 세션 시작 훅이 자동으로 이 명령을 돌려 그 결과를 에이전트의 첫 컨텍스트로 주입하는 경우다. 후자가 핵심 용도이며, 이 덕분에 에이전트는 세션이 바뀌어도 "지금 무슨 작업 중인지"를 잃지 않는다.

## 요청 / 응답

호출자는 어느 디렉터리를 기준으로 삼을지(생략하면 현재 디렉터리), 그리고 무엇까지 보여 줄지(진행 중인 것만 / 다음에 할 것 / 전부, 기본은 전부)를 함께 넘긴다. 디렉터리 경로는 절대 경로로 정규화되어 등록된 프로젝트의 경로와 정확히 맞춰진다.

돌려주는 것은 사람이 그대로 읽을 수 있는 요약 문장 묶음이다. 활성 플랜과 그 하위 구조, 진행 중인 태스크가 자연어 형태로 담긴다. 추가로, 한 프로젝트에 활성 플랜이 둘 이상 떠 있으면 별도 경고가 함께 나온다 — PDD 원칙상 프로젝트당 활성 플랜은 하나가 권장(전환기에는 둘까지 허용)이기 때문이며, 이 경고는 본문 출력과 분리되어 표시되어 훅이 따로 걸러 읽을 수 있다.

## 권한 / 제약

로컬 데몬이 떠 있어야 동작한다. 출력 대상은 호출 디렉터리가 매핑된 단일 프로젝트로 한정되며, 디렉터리가 어떤 프로젝트에도 등록되어 있지 않으면 보여 줄 요약이 없다. 활성 플랜 개수 경고는 차단이 아니라 안내성 신호다(작업을 막지 않는다).

## provenance

`cli/src/main.rs`의 `Dashboard` 명령 정의와 처리부에서 확인했다. 처리부는 작업 디렉터리를 절대 경로로 정규화한 뒤 데몬의 대시보드 조회 경로를 호출하고, 응답 안의 요약 문자열을 그대로 출력하며, 활성 플랜 다수 경고를 별도 스트림으로 내보낸다. 실제 요약 데이터를 만드는 주체는 데몬(`component.daemon`)이고, CLI(`component.cli`)는 이를 호출·렌더링하는 얇은 표면이다.

## 미확정 (OPEN)
- [ ] OPEN: `show=active|next|all` 각 모드가 데몬 측에서 정확히 어떤 항목을 포함/제외하는지(예: blocked 태스크 포함 여부)는 데몬 라우터 정독으로 확정 필요.
