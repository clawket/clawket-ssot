---
id: capability.import-export-plan
kind: Capability
title: 플랜·지식 가져오기/내보내기
purpose: "마크다운으로 작성한 플랜을 가져오고 플랜·지식을 md/json/yaml로 내보내, 외부 도구·파일과 오가되 DB를 단일 진실 원천으로 유지한다."
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: domain.work-management, type: relates-to, note: "플랜 가져오기/내보내기가 속한 작업 관리 도메인" }
  - { to: concept.plan, type: relates-to, note: "import/export 대상 플랜" }
  - { to: concept.execution-envelope, type: relates-to, note: "strict 모드에서 라인 단위로 검증·영속되는 19필드 실행 봉투" }
  - { to: concept.knowledge, type: relates-to, note: "docs/와 가져오기·내보내기되는 지식 항목" }
  - { to: endpoint.cli-plan, type: calls, note: "plan import/export 하위 명령" }
  - { to: endpoint.cli-knowledge, type: calls, note: "knowledge import/export 하위 명령" }
impacts:
  - concept.plan
  - concept.knowledge
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 사용자가 할 수 있는 일

플랜과 지식은 파일로 주고받을 수 있다. 마크다운으로 써둔 플랜을 그대로 가져와 Clawket의 플랜·유닛·사이클·태스크로 만들고, 반대로 현재 플랜이나 지식을 마크다운/JSON/YAML 파일로 내보낸다. 다만 핵심 원칙은 "DB가 단일 진실 원천"이라는 것 — 내보낸 마크다운은 손으로 고치는 문서가 아니라 DB 상태를 비춘 생성물(view)이며, 봉투를 수정한 뒤 스냅샷을 다시 만들거나 다른 도구에 플랜을 넘길 때 쓴다.

## 행위

- **플랜 가져오기(loose)**: 기본 모드로, 봉투 불릿과 의존성 그래프를 무시하고 마크다운에서 플랜 골격만 읽어 엔티티를 만든다. 가져오기 전 미리보기(dry-run)와 프로젝트 자동 생성(없으면)을 지원한다.
- **플랜 가져오기(strict)**: 19필드 실행 봉투 파서로, 봉투와 의존성을 라인 단위로 검증하고 봉투·의존관계 레코드까지 영속한다. 내보낸 마크다운과의 왕복(round-trip) 일치를 보장하는 모드다.
- **플랜 내보내기**: 마크다운(기본)·JSON(정식 구조)·YAML 중 한 형식으로 내보낸다. 첨부 지식을 부록으로 포함하는 옵션이 있으나 기본은 꺼져 있어 크기를 예측 가능하게 둔다.
- **지식 가져오기/내보내기**: 작업 디렉터리의 docs/ 파일을 지식 항목으로 가져오고, 지식 항목을 docs/로 내보낸다(플랜·유닛 단위로 범위 지정 가능, 미리보기 지원).

## 시스템 흐름

플랜 import/export는 `endpoint.cli-plan`의 import·export 하위 명령으로, 지식 import/export는 `endpoint.cli-knowledge`의 동일 하위 명령으로 들어간다. CLI가 파일을 파싱하거나 직렬화하고, 엔티티 생성·조회는 데몬 API로 위임한다. strict 가져오기는 봉투·의존 그래프를 데몬에 영속하므로 단순 텍스트 파싱이 아니라 검증을 동반한 쓰기 경로다.

## 어디에 구현되어 있나

CLI 명령 정의는 `cli/src/main.rs`에 있다. plan import 명령에 dry-run·project 자동생성·strict 플래그가, plan export 명령에 format(md/json/yaml)·include-knowledge 옵션이 선언되며, knowledge import/export는 docs/ 디렉터리 스캔·쓰기를 cwd·plan·unit 범위로 다룬다.

## 미확정 (OPEN)
- [ ] OPEN: strict 파서의 19필드 검증 규칙과 의존성 그래프 영속 경로(데몬/봉투 모듈) 정밀 확인 필요
