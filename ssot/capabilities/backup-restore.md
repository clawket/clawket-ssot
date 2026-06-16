---
id: capability.backup-restore
kind: Capability
title: 백업·복원
definition: 사람이나 LLM 에이전트가 Clawket의 모든 데이터(데이터베이스와 첨부된 지식 항목)를 휴대 가능한 tar.gz 압축본으로 내보내고, 그 압축본을 다른 기기로 옮기거나 보관했다가 현재 데이터를 통째로 교체하거나 기존 데이터 위에 겹쳐 얹는(merge) 방식으로 되돌리는 일이다.
purpose: 로컬에만 존재하는 작업 상태·지식을 휴대 가능한 단일 아카이브로 보존하고, 기기 이전·오프사이트 백업·복구 시 교체 또는 병합으로 복원한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
  - component.sqlite-storage
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.cli-backup-restore, type: calls, note: "backup 으로 내보내고 restore(기본 교체, --merge 병합, --dry-run 미리보기)로 되돌린다" }
  - { to: concept.knowledge, type: reads, note: "첨부된 지식 항목이 DB와 함께 아카이브에 포함된다" }
  - { to: component.sqlite-storage, type: relates-to, note: "백업·복원의 대상은 로컬 SQLite 데이터베이스 전체다" }
impacts:
  - concept.project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

Clawket의 모든 상태는 로컬에만 있다 — 클라우드에 동기화되지 않는다. 그래서 다른 기기로 작업을 옮기거나, 만일을 대비해 오프사이트 백업을 두거나, 사고 후 복구하려면 직접 내보내고 되돌리는 수단이 필요하다. 백업은 데이터베이스와 거기에 첨부된 지식 항목까지 하나의 tar.gz 압축본으로 묶어내며, 전체 또는 특정 프로젝트 하나만 골라 내보낼 수 있다.

복원에는 두 가지 의도가 있다. 기본은 "교체" — 현재 데이터베이스를 아카이브 내용으로 대체한다. `--merge`를 쓰면 아카이브를 기존 데이터 위에 겹쳐 얹어, 지금 데이터를 지우지 않고 합친다. 적용 전 무엇이 바뀌는지 먼저 보고 싶으면 미리보기(dry-run)로 변경 없이 확인할 수 있다.

## 행위

- 전체 또는 특정 프로젝트의 데이터(DB + 첨부 지식)를 tar.gz 아카이브로 내보낸다(출력 경로 미지정 시 타임스탬프가 붙은 기본 파일명).
- 아카이브를 교체 방식으로 복원한다(기본) — 현재 DB를 아카이브 내용으로 대체.
- 아카이브를 병합 방식으로 복원한다(`--merge`) — 기존 데이터 위에 겹쳐 얹어 합친다.
- 적용 없이 변경 내용만 미리 본다(`--dry-run`).

## 시스템 흐름

사용자나 에이전트가 백업을 요청하면 CLI가 로컬 데몬에 내보내기를 요청하고, 데몬이 SQLite 데이터베이스와 첨부 지식을 모아 tar.gz 아카이브로 직렬화한다. 복원은 그 역방향으로, 아카이브를 데몬에 전달하면 데몬이 교체 또는 병합 정책에 따라 로컬 저장소에 반영한다. 미리보기 모드에서는 데몬이 실제 쓰기 없이 어떤 변경이 일어날지만 계산해 보고한다.

## 어디에 구현되어 있나

CLI의 backup·restore 명령 정의는 `cli/src/main.rs`에 있고(교체/`--merge`/`--dry-run` 옵션 포함), 실제 직렬화·적용은 데몬의 백업·복원 경로를 통해 로컬 SQLite를 대상으로 수행된다.

## 미확정 (OPEN)
- [ ] OPEN: 병합(--merge) 복원 시 동일 ID 충돌(같은 프로젝트·태스크가 양쪽에 있을 때)의 해소 규칙은 데몬 측 복원 경로에서 추가 확인 필요.
- [ ] OPEN: 아카이브에 포함되는 정확한 범위(감사 로그·임베딩 벡터 포함 여부)는 코드에서 교차 확인 필요.
