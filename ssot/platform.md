---
id: platform.claude-code-plugin
kind: Platform
title: Clawket — Claude Code 플러그인
purpose: LLM 코딩 에이전트와 사람이 같은 작업 상태를 공유하며 구조화된 작업·지식을 로컬에서 관리하게 한다
servesPersona: [persona.llm-coding-agent, persona.human-operator]
value: 로컬 우선(local-first)·에이전트-사람 공유 상태·훅 강제로 LLM 주도 개발을 추적·통제
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:platform
---

## 무엇인가

Clawket은 **LLM-네이티브 작업 관리 도구 + 로컬 RAG 캐시**다. Claude Code에 설치되는 플러그인은 훅·프롬프트·스킬·MCP 등록만 담는 얇은 셸이고, 실제 런타임(작업·지식 저장, 검색, 데몬)은 별도 바이너리(Rust CLI·데몬, 웹/데스크탑 앱)에 위임한다. 모든 상태는 **로컬 SQLite**에 저장되고 데몬은 루프백·유닉스 소켓에만 바인딩하며 임베딩도 온디바이스라 어떤 데이터도 외부로 나가지 않는다(local-first).

## 누구를 위한 것인가

- **LLM 코딩 에이전트(Claude Code)**: CLI·MCP로 작업 상태를 읽고 쓰는 주체. 활성 태스크 없이는 변경 작업이 훅에 의해 차단된다.
- **사람 운영자**: 웹·데스크탑 대시보드(Board·Backlog·Timeline·Wiki·Summary)로 에이전트와 **동일한 상태**를 보고 관리한다.

둘은 Jira+Confluence를 대체하는 구조화된 상태 계층 — **Project → Plan → Unit → Cycle → Task** 워크플로우 — 위에서 협업한다.

## 서비스 영역

- **작업 관리**: 프로젝트·플랜·유닛·사이클·태스크로 LLM 주도 작업을 추적.
- **지식·로컬 RAG**: 버전 관리 지식 항목을 온디바이스 임베딩으로 FTS5+벡터 하이브리드 검색, 세션 간 컨텍스트 복원.
- **훅 강제(가드레일)**: Claude Code 훅 이벤트마다 활성 태스크·플랜·사이클 상태와 파괴적 명령을 검사해 허용/차단.
- **PDD 검증·수렴 루프**: 시나리오↔코드 대조 검증으로 결함·시나리오오류를 0으로 수렴.

## 접근 매트릭스 (역할 → 서비스)

| 역할 | 접근 경로 | 비고 |
|------|----------|------|
| LLM 코딩 에이전트 | CLI · MCP 도구 | 활성 태스크 없으면 변경 차단(훅) |
| 사람 운영자 | 웹·데스크탑 대시보드 | 읽기·관리(DnD), 같은 SQLite 상태 |

두 역할 모두 동일한 로컬 SQLite 상태를 공유한다 — 한쪽의 변경이 다른 쪽에 즉시 반영된다.

## 기술 스택 (제품 맥락)

- 플러그인: Claude Code 플러그인(훅·프롬프트·스킬·MCP, TS/JS)
- 런타임: Rust CLI + 데몬(로컬 SQLite, FTS5 + 온디바이스 벡터 임베딩, MCP 서버)
- UI: 웹(React) · 데스크탑 앱

## 미확정 (OPEN)

- [ ] OPEN: 코드/문서에서 추론(confidence: inferred) — 담당자 검증 후 confidence 승격.
