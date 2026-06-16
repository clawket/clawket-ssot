---
id: component.cli
kind: SystemComponent
title: clawket CLI (Rust 바이너리)
definition: 단일 정적 Rust 바이너리로 작업관리 명령 전체와 RAG 별칭, 진단·온보딩, 그리고 임베디드 MCP stdio 서버(clawket mcp)를 제공하며 릴리스에서 다운로드되는 LLM·사람의 명령줄 진입점.
purpose: LLM 코딩 에이전트(주로 CLI를 통해)와 사람이 동일한 작업 상태를 읽고 쓰는 표면을 제공하기 위해 존재한다. 프로젝트·플랜·유닛·사이클·태스크·지식·실행기록 등 모든 작업관리 동작을 명령으로 노출하고, 같은 바이너리 안에 MCP 서버를 내장해 Claude Code가 read-only로 컨텍스트를 당겨오게 한다. 단일 정적 바이너리로 배포해 Rust 툴체인 없이 설치·실행되도록 한다.
realizedBy:
  - domain.work-management
  - capability.manage-task-lifecycle
  - capability.rag-pull
implementedIn:
  - cli/src
  - cli/Cargo.toml
  - cli/src/main.rs
  - cli/src/mcp.rs
dependsOn:
  - component.daemon
  - component.mcp-server
consumesApi:
  - endpoint.daemon-health
  - endpoint.daemon-events
  - endpoint.daemon-search
providesApi:
  - endpoint.cli-dashboard
  - endpoint.cli-daemon
  - endpoint.cli-mcp
  - endpoint.cli-doctor
  - endpoint.cli-verify
  - endpoint.cli-init
  - endpoint.cli-project
  - endpoint.cli-plan
  - endpoint.cli-unit
  - endpoint.cli-cycle
  - endpoint.cli-task
  - endpoint.cli-knowledge
  - endpoint.cli-run
  - endpoint.cli-comment
  - endpoint.cli-question
  - endpoint.cli-config
  - endpoint.cli-discover-loop
  - endpoint.cli-watch
  - endpoint.cli-replay
  - endpoint.cli-backup-restore
  - endpoint.cli-migrate
  - endpoint.cli-update-versioncheck
  - endpoint.cli-web-views
  - endpoint.cli-rag-aliases
  - endpoint.cli-completions
integratesWith:
  - integration.github-releases
  - integration.mcp
impacts:
  - persona.llm-coding-agent
  - persona.human-operator
  - domain.work-management
  - domain.knowledge-rag
relatesTo:
  - to: component.mcp-server
    type: contains
    note: MCP stdio 서버가 CLI 바이너리 안에 임베드되어 clawket mcp로 기동됨
  - to: component.daemon
    type: depends-on
    note: 모든 상태 읽기·쓰기는 데몬 HTTP API를 통해 처리
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 책임

clawket 명령으로 호출되는 단일 정적 Rust 바이너리다. Clawket의 작업관리 동작 전체 — 프로젝트 등록·관리, 플랜 작성·승인, 유닛·사이클·태스크의 라이프사이클, 지식 CRUD·검색, 실행기록·코멘트·질문, 백업·복원, 마이그레이션, 업데이트 — 의 명령줄 표면이 여기 있다. 더해 검증 루프 자동화(discover-loop), 실시간 관찰(watch)·리플레이, 진단(doctor)·스모크 검증(verify)·온보딩(init), RAG 별칭 명령, 셸 자동완성까지 제공한다.

이 바이너리에는 MCP stdio 서버(`component.mcp-server`)가 임베드되어 있어 `clawket mcp` 서브커맨드로 기동된다. 즉 CLI는 사람·에이전트의 명령줄 진입점이면서, 동시에 Claude Code가 read-only로 컨텍스트를 당겨오는 MCP 서버의 호스트이기도 하다.

## 경계와 의존

CLI 자체는 작업 상태를 직접 보관하지 않는다. 모든 읽기·쓰기는 로컬 데몬(`component.daemon`)의 HTTP API를 통해 이뤄지며, 데몬이 SQLite를 소유한다. 따라서 CLI는 데몬에 의존하고, 명령 실행 전 데몬이 떠 있는지 확인·필요 시 기동하는 흐름을 거친다. 바이너리 자체는 Rust 툴체인 없이 설치되도록 GitHub Releases에서 받아온다(`integration.github-releases`).

제공하는 명령(엔드포인트)은 위 frontmatter의 providesApi 목록과 1:1로 대응한다. 소비하는 데몬 API로는 헬스(`endpoint.daemon-health`), 이벤트 스트림(`endpoint.daemon-events` — watch가 구독), 검색(`endpoint.daemon-search` — search 계열·RAG 별칭이 사용)이 있다.

## 통신 패턴

명령 실행은 CLI → 데몬 HTTP API 호출로 이뤄지고, 데몬은 SQLite를 갱신한 뒤 결과를 돌려준다. 실시간 관찰 명령은 데몬의 SSE 이벤트 버스를 구독해 task/cycle/run 변경을 흘려보낸다. MCP 서브커맨드로 기동되면 stdio로 Claude Code와 말하고, 내부적으로는 데몬 포트를 자동 발견해 데몬 HTTP API(검색 등)를 호출한다.

## 하위 서브패키지 (책임 단위)

- 작업관리 명령군(project/plan/unit/cycle/task/knowledge/run/comment/question/config): 각 엔티티의 CRUD·라이프사이클.
- 검증·자동화(discover-loop, watch, replay): PDD 검증 루프 구동과 실행 관찰.
- 운영·진단(daemon, doctor, verify, migrate, update/version-check, backup/restore, completions, init): 데몬 제어와 설치·건강·온보딩.
- RAG 별칭(find-similar/get-task-context/get-recent-decisions): 의미 검색 단축 명령.
- 임베디드 MCP 서버(`component.mcp-server`): clawket mcp로 노출되는 read-only 지식 도구 호스트.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 미기동 시 CLI가 자동 기동을 시도하는 범위·실패 처리의 정확한 경계는 endpoint.cli-daemon / component.daemon 노드에 위임.
