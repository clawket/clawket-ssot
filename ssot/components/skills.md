---
id: component.skills
kind: SystemComponent
title: 번들 스킬 7종
definition: 플러그인이 함께 배포하는 7개의 스킬 묶음으로, 작업 대시보드 표면 1종과 PDD(시나리오·플랜·검증·정제·결함수정) 워크플로우 6종으로 이루어진다. 각 스킬은 Claude Code의 슬래시 명령(또는 기본 대시보드 표면)으로 진입해 작업관리·검증 워크플로우를 안내하는 진입점이다.
purpose: "사람과 LLM 에이전트가 Clawket의 작업관리와 PDD 검증 루프를 슬래시 명령 단위로 시작할 수 있게 하는 워크플로우 진입점 묶음. 각 스킬은 해당 단계의 절차·규칙·산출물 형식을 담은 안내서다."
realizedBy:
  - domain.work-management
  - domain.pdd-verification
  - capability.run-verification-loop
implementedIn:
  - clawket/skills
  - clawket/.claude-plugin/plugin.json
dependsOn:
  - component.cli
  - component.plugin-shell
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.llm-coding-agent
  - persona.human-operator
  - domain.pdd-verification
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: component.plugin-shell
    type: belongs-to
    note: 스킬 파일들은 플러그인 셸 레포에 동봉되어 plugin.json::skillsList로 등록된다.
  - to: domain.pdd-verification
    type: realizes
    note: 7종 중 6종이 PDD 검증·수렴 루프의 각 단계를 직접 구현하는 워크플로우다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 분기/의도 불명은 OPEN. -->

## 책임

이 구성 요소는 Clawket의 워크플로우를 "어떻게 시작하고 어떤 순서로 진행하는가"를 안내하는 7개의 스킬 묶음이다. 각 스킬은 Claude Code에 등록되어, 사용자가 슬래시 명령을 입력하거나(6종) 기본 대시보드 표면을 통해(1종) 해당 워크플로우로 진입하게 한다. 스킬 자체는 실행 로직이 아니라, 그 단계에서 무엇을·어떤 규칙으로·어떤 산출물 형식으로 해야 하는지를 담은 안내서다 — 실제 상태 변경은 CLI와 데몬이 수행한다.

일곱 스킬은 두 갈래로 나뉜다. 하나는 작업 상태를 보고 태스크를 시작·완료·생성하는 대시보드 표면(슬래시 없음)이고, 나머지 여섯은 PDD(시나리오 주도 검증) 워크플로우의 각 단계다: 플랜·유닛 사전 설계, 원자적 시나리오 작성, 검증 배치 디스패치, 검증 라운드 전체 실행(3-way 수렴 판정 포함), 시나리오 오류 정제, 결함 수정 태스크 등록·적용.

## 경계와 의존

스킬 파일들은 플러그인 셸(`component.plugin-shell`) 레포에 동봉되며, 플러그인 매니페스트(`plugin.json`)의 스킬 목록에 등록되어 Claude Code가 인식한다. 스킬이 안내하는 절차를 실제로 수행할 때는 CLI(`component.cli`) 명령(예: 검증 루프 자동화 명령)과, 그를 통해 데몬·저장소에 도달하는 경로에 의존한다. 즉 스킬은 "절차 명세 + 진입점"이고, 그 절차의 집행자는 CLI·데몬이다.

대시보드 표면 스킬은 작업 라이프사이클 게이트(활성 태스크 없으면 변경 차단 등)를 거쳐 시작·완료·생성을 라우팅한다. PDD 6종은 시나리오→플랜→검증→정제/수정으로 이어지는 수렴 루프의 단계별 역할을 나눠 맡는다.

## 통신 패턴

스킬은 Claude Code의 스킬 메커니즘을 통해 로드·호출된다. 외부 네트워크 통신은 스킬 자체에 없으며, 스킬이 지시하는 작업이 CLI/MCP/데몬을 거쳐 로컬에서 처리된다. 검증 배치 스킬은 다수의 서브에이전트를 배치 단위로 스폰하고, 그 산출물(7필드 TSV 근거)을 다중 워커로 Clawket 태스크에 동기화하도록 안내한다.

## 하위 서브패키지 (책임 단위)

- clawket-dashboard: 작업 상태 조회·태스크 시작/완료/생성을 라이프사이클 게이트로 라우팅하는 기본 대시보드 표면(슬래시 없음).
- clawket-plan-design: Done 명제 고정·시나리오의 유닛 분해·유닛 의존 그래프 선언·수렴 조건 설정.
- clawket-scenario-author: As-a/I-want/So-that + Given/When/Then 형식의 원자적 시나리오 작성.
- clawket-verify-batch: 서브에이전트 배치 디스패치 + 배치별 7필드 TSV 근거 + 다중 워커 벌크 동기화.
- clawket-verify-loop: 한 검증 라운드를 끝까지 실행하고 3-way 수렴(defect/scenario_error/converged)을 판정.
- clawket-scenario-refine: scenario_error 행에 대한 분할/의도재정의/삭제 3-way 처리.
- clawket-defect-fix: defect 행마다 결함해결 플랜의 Round R Unit 아래 수정 태스크 등록·적용.

## 미확정 (OPEN)
- [ ] OPEN: 각 스킬의 슬래시 명령이 호출하는 CLI 서브커맨드와의 정확한 1:1 매핑은 스킬별 SKILL.md에서 추가 추적.
