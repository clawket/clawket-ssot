---
id: screen.docs-cli
kind: Screen
title: CLI 레퍼런스 화면(CLI Reference)
definition: /docs/cli 경로의 영문 문서 화면. clawket CLI가 로컬 데몬과 유닉스 소켓(TCP 루프백 폴백)으로 통신하는 단일 정적 Rust 바이너리임을 전제로, --help가 정본임을 강조하고 최상위 서브커맨드 맵·전체 워크 예시·RAG 검색·doctor 진단·ULID/티켓번호 식별자 체계를 내비게이션 형태로 안내한다.
purpose: CLI로 Clawket을 다루려는 사람이 어떤 서브커맨드가 무엇을 하는지 한눈에 파악하고, 정확한 플래그는 --help로 확인하도록 길잡이를 제공한다.
servesPersona: [persona.human-operator, persona.llm-coding-agent]
realizedBy: [component.landing-site]
implementedIn: [landing/src/pages/docs/CLI.tsx]
consumesApi: []
relatesTo:
  - { to: component.cli, type: relates-to, note: "clawket 단일 정적 Rust 바이너리의 서브커맨드 표면을 안내" }
  - { to: capability.rag-pull, type: relates-to, note: "task/knowledge search 하이브리드(FTS5+벡터) 검색 안내" }
  - { to: capability.wiki-knowledge, type: relates-to, note: "knowledge 서브커맨드와 wiki-tree 안내" }
  - { to: concept.ticket-number, type: relates-to, note: "ULID와 병행하는 사람용 티켓번호(CK-42 등) 식별자 체계 설명" }
  - { to: endpoint.cli-doctor, type: relates-to, note: "clawket doctor 진단(경로분리 불변식·/health·버전·스킬) 안내" }
  - { to: endpoint.cli-task, type: relates-to, note: "clawket task 서브커맨드 안내" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

CLI 사용자를 위한 길잡이 문서다. 핵심 메시지는 "이 페이지는 모든 플래그를 나열한 레퍼런스가 아니라 내비게이션 맵이며, 정확한 정본은 언제나 --help"라는 것이다. 방문자는 여기서 어떤 서브커맨드가 무엇을 담당하는지 큰 그림을 잡고, 실제 옵션은 clawket --help / clawket <sub> --help / clawket <sub> <action> --help 로 확인한다.

## UI 요소 / 입력 필드

- 머리말: clawket이 로컬 clawketd 데몬과 유닉스 소켓(TCP 루프백 폴백)으로 통신하는 단일 정적 Rust 실행 파일이라는 설명과, 3단계 --help 사용 규율 코드 블록.
- "최상위 서브커맨드": project(등록·목록·cwd 바인딩), plan(생성·승인·목록·완료 — approve가 active 게이트), unit(생성·목록, 상태 없는 그룹핑), cycle(--unit 필요, activate·complete, 스프린트), task(실제로 작업하는 엔티티 — 생성·상태변경·근거첨부·코멘트·목록), knowledge(생성·검색·정리, MCP로 LLM이 조회하는 RAG 계층, wiki-tree로 계층 렌더), run(태스크별 실행 기록, 보통 훅이 자동 생성), events replay(감사로그를 유한 SSE 스트림으로 재생), comment·question(태스크 토론), dashboard(cwd의 active 플랜/유닛/사이클/태스크 출력), daemon(start/stop/status), mcp(내장 MCP stdio 서버 — Claude Code용, 대화형 아님), doctor(설치 진단), completions(셸 완성 스크립트 생성).
- "워크 예시": 프로젝트 생성 → 플랜 생성·승인 → 유닛 생성 → 사이클 생성(--unit)·활성화 → 태스크 생성(봉투 필드 + --tier)·in_progress → (작업·PostToolUse 기록) → done(--evidence, 데몬이 EVIDENCE_REQUIRED 강제)까지의 전체 명령 흐름.
- "RAG 검색": 태스크·지식이 온디바이스 다국어 모델로 생성·갱신 시 임베딩되며, CLI가 웹 위키·MCP와 같은 하이브리드(FTS5+벡터) 검색 엔드포인트를 노출한다는 설명과 task search / knowledge search 예시.
- "Doctor": 문제 시 먼저 doctor를 돌리라는 안내 — 경로 분리 불변식(사용자 데이터가 ~/.claude/plugins/ 아래 있으면 안 됨), 데몬 /health 핑, 핀된 컴포넌트와 바이너리 버전 일치, 스킬 파일 존재를 점검하고, 비정상 종료코드는 실제 문제(예: 경로 겹침 시 데몬 기동 거부)이며 CLAWKET_ALLOW_PLUGIN_OVERLAP=1은 데이터 손실 위험을 감수하는 우회라는 설명.
- "식별자": 모든 엔티티가 안정 참조용 ULID(예: TASK-01KS…)와 대화/커밋/PR용 짧은 사람용 티켓번호(CK-42 또는 프로젝트 접두 ABC-7)를 병행하며 CLI가 둘 다 받는다는 설명.

입력 필드는 없고 코드 블록은 읽기/복사용이다.

## 표시 데이터 / 호출 API

하드코딩된 영문 설명과 코드 예시다. 화면 자체는 데몬·DB를 호출하지 않는다(consumesApi 없음). 본문이 설명하는 명령은 사용자가 직접 실행할 때 CLI를 통해 데몬과 통신한다.

## 상태 / 엣지케이스

- 정적 문서라 로딩/에러 상태가 없다.
- 진입 시 자기 SEO 메타데이터를 직접 설정한다.

## 미확정 (OPEN)
- 없음
