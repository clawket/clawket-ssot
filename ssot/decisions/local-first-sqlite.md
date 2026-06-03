---
id: decision.local-first-sqlite
kind: Decision
title: 모든 상태를 로컬 SQLite 한 파일에 두고 데몬은 멀티테넌트로 노출하지 않는다
purpose: "여러 표면(CLI·웹·MCP)이 공유하는 작업 상태를 어디에 어떻게 보관할지 — 외부 서버/멀티테넌트 DB로 갈지, 사용자 기기 안에 가둘지"
definition: "데몬은 단일 SQLite 파일을 모든 상태의 단일 진실 저장소로 삼고, 루프백(localhost) 외 주소로는 기본적으로 바인딩을 거부해 한 사용자의 로컬 기기 밖으로 데이터가 나가거나 다수 사용자가 공유하는 형태로 노출되지 않게 한다."
relatesTo:
  - to: component.sqlite-storage
    type: relates-to
    note: 이 결정이 지정하는 단일 진실 저장소 — 모든 작업 상태가 한 파일에 모인다
  - to: component.daemon
    type: relates-to
    note: SQLite 파일을 소유하고 루프백 전용으로 바인딩하는 주체
  - to: platform.local-first
    type: relates-to
    note: 데이터가 기기 밖으로 나가지 않는 로컬 우선 동작의 토대
  - to: invariant.path-separation-xdg
    type: relates-to
    note: 저장소 파일 위치를 사용자 데이터 디렉터리로 고정하는 경로 규약
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

<!-- Decision은 append-only. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 supersedes로 잇는다. -->

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다. (스냅샷 원칙 — 히스토리는 여기와 git에만)

## 맥락 (Context)

Clawket은 사람과 LLM 에이전트가 같은 작업 상태(프로젝트·플랜·태스크·지식 등)를 동시에 읽고 쓰는 도구다. 이 공유 상태를 어디에 둘지가 근본 갈림길이었다. 클라우드 데이터베이스나 다수 사용자가 함께 쓰는 서버를 두는 길은 어디서나 접근·협업이 가능하다는 장점이 있지만, 사용자의 개발 작업 내역·코드 컨텍스트·의사결정 기록이 기기 밖 외부로 흘러나간다. Clawket이 다루는 데이터는 본질적으로 한 사람의 로컬 개발 작업에 묶여 있고, 외부 노출 시 프라이버시·신뢰 비용이 크다.

## 결정 (Decision)

상태를 사용자 기기 안의 단일 SQLite 파일에 모으고, 데몬은 그 파일을 단독으로 소유한다. 데몬은 네트워크에 열 때도 루프백 주소(localhost) 와 유닉스 소켓에만 붙고, 그 밖의 주소(LAN·공인 IP·전체 인터페이스)로의 바인딩은 기본적으로 거부한다. 외부 노출이 정말 필요한 운영자만 명시적으로 옵트인해야 풀린다. 즉 "로컬 우선, 멀티테넌트 미설계"를 제품의 전제로 못 박았다.

## 근거와 결과 (Consequences)

- **단일 진실 저장소**: CLI·웹·MCP가 제각기 상태를 갖지 않고 한 파일을 공유하므로, 어느 표면에서 본 상태든 동일하다. 표면 간 상태 불일치(single source of truth 위반)가 구조적으로 차단된다.
- **프라이버시·신뢰**: 데이터가 기기 밖으로 나가지 않으므로 외부 유출 경로 자체가 없다. 로컬 우선 동작과 임베딩의 온디바이스 실행이 이 결정 위에서 성립한다.
- **부수 제약**: 멀티테넌트로 설계하지 않았으므로 SQLite 파일을 여러 사용자가 동시 공유하는 형태는 지원 대상이 아니다. 외부에서 접근시키려는 시도(비루프백 바인딩)는 명시적 옵트인을 요구해 사고로 공개되는 것을 막는다.
- **백업·이동**: 상태가 한 파일에 모여 있어 백업·복원·기기 이전이 파일 단위로 단순해진다.

provenance: 루프백 외 바인딩 거부 가드 — `daemon/src/main.rs:129-140`(PUBLIC_BIND_NOT_ALLOWED, 옵트인 환경변수로만 해제). 단일 SQLite 파일을 마이그레이션 러너가 트랜잭션 안에서 다루는 저장 모델 — `daemon/src/db.rs`(SqlitePool, WAL 모드, `SCHEMA_VERSION_MAX`).
