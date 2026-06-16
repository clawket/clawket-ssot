---
id: endpoint.get-recent-decisions
kind: Endpoint
title: clawket get-recent-decisions (CLI 명령 정의)
definition: 프로젝트의 결정(decision) 유형 지식 항목을 생성 시간 역순으로 추려 반환하는 최상위 RAG 별칭 명령.
realizedBy: [component.cli]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.knowledge, type: reads, note: "type=decision 인 지식 항목만 골라 최신순으로 돌려준다" }
  - { to: concept.project, type: reads, note: "project 옵션으로 특정 프로젝트의 결정만 좁힌다" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

LLM 코딩 에이전트가 새 세션을 시작할 때 "이 프로젝트에서 그동안 어떤 결정이 내려졌나"를 빠르게 회수하기 위해 쓰는 최상위 단축 명령이다. 위키 지식 항목 중 의사결정 기록(type=decision)으로 분류된 것만 골라 최신 것부터 보여 준다. clawket knowledge 검색의 한 갈래를 별칭으로 끌어올린 셈으로, 검색어 없이 "최근 결정 모음"을 그대로 받아 보는 용도다.

## 요청 / 응답

요청으로는 결정을 어느 프로젝트로 좁힐지(프로젝트 범위)와 몇 건까지 받을지(상한, 기본 10건)를 줄 수 있다. 응답으로는 결정 지식 항목 목록이 돌아오며, 각 항목은 제목·내용·소속 프로젝트·생성 시각 등 지식 항목이 가진 정보를 담는다. 목록은 생성 시각이 새로운 것부터 정렬되고, 요청한 상한 수만큼 잘라서 반환한다.

## 권한 / 제약

데몬이 살아 있어야 동작한다(데이터는 데몬이 소유한 로컬 SQLite에 있고, 명령은 데몬 HTTP API를 통해 지식 목록을 읽어 온다). 결정만 반환하도록 type=decision 으로 고정 필터가 걸려 있어 다른 유형의 지식은 결과에 섞이지 않는다. 정렬·상한 적용은 명령 측에서 수행한다(데몬이 돌려준 전체 결정 목록을 최신순으로 정렬한 뒤 상한 건수로 자른다). 읽기 전용이라 데이터를 바꾸지 않는다.

## provenance

명령 정의와 처리 로직은 cli/src/main.rs 에 있다(명령 enum 정의 331-340행, 실행 분기 5179-5193행). 이 명령은 동일 기능을 MCP 도구로도 노출하는 clawket_get_recent_decisions(endpoint.mcp-get-recent-decisions)와 짝을 이루며, CLI 측 별칭 묶음 endpoint.cli-rag-aliases 의 한 갈래로도 정리되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: 프로젝트 범위를 생략했을 때(전체 프로젝트 대상) 데몬이 모든 프로젝트의 결정을 합쳐 돌려주는지, 활성 프로젝트로 암묵 한정되는지 데몬 라우트 정독 필요.
