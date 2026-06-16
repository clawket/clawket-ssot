---
id: endpoint.knowledge-list
kind: Endpoint
title: 지식 엔트리 목록·생성 (데몬 GET·POST /knowledge)
definition: 위키성 지식 엔트리를 조건별로 조회하고, 새 엔트리를 만들면서 즉시 임베딩까지 동기적으로 처리하는 데몬 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/knowledge.rs]
relatesTo:
  - { to: concept.knowledge, type: reads, note: "task/unit/plan/project/type 조건으로 지식 목록 조회" }
  - { to: concept.knowledge, type: mutates, note: "지식 엔트리 생성 + 동기 임베딩(실패 시 롤백)" }
  - { to: endpoint.daemon-search, type: relates-to, note: "생성 시 임베딩이 검색의 read-your-write 일관성을 보장" }
governedBy: [invariant.local-only-no-egress]
impacts: [concept.knowledge]
consumedBy: [endpoint.cli-knowledge, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
위키처럼 쌓이는 지식 엔트리(결정·노트·근거·참조 등)를 조건에 맞게 목록으로 가져오고, 새 엔트리를 작성한다. 작성된 엔트리는 곧바로 의미 검색용 임베딩(벡터)으로 변환되어 저장되므로, 만들자마자 검색에 잡힌다. 사람이 위키 화면에서 문서를 새로 쓰거나, 에이전트가 결정·근거를 기록할 때 이 경로를 탄다.

## 요청 / 응답
- 목록 조회는 어느 태스크·유닛·플랜에 붙은 것인지, 어느 프로젝트 범위인지, 어떤 유형(type)인지로 좁혀서 요청할 수 있다. 응답은 조건에 맞는 지식 엔트리 목록이다. 단, 어디에도 붙지 않은 "떠도는" 엔트리는 사용자 범위로 보아 프로젝트 필터와 무관하게 노출된다.
- 생성 요청은 유형·제목과 (선택적으로) 본문·본문형식·부모 엔트리, 그리고 붙일 대상(태스크/유닛/플랜)을 보낸다. 응답은 만들어진 엔트리 한 건이다.
- 본문이 비어있지 않으면 생성과 같은 흐름에서 임베딩을 만든다. 이 임베딩이 실패하면 방금 만든 엔트리를 되돌려(삭제) "임베딩 실패"로 응답한다 — 즉 절반만 저장된 상태를 남기지 않는다.

## 권한 / 제약
로컬 데몬 내부 API로 외부 인증은 없다. 임베딩은 온디바이스(기기 안)에서 수행되어 어떤 데이터도 외부로 나가지 않는다(로컬 전용·무유출 불변식). 생성은 임베딩 성공과 원자적으로 묶여, 검색에서 방금 쓴 것을 바로 읽을 수 있는 일관성을 보장한다. 프로젝트 필터는 엔트리가 붙은 태스크/유닛/플랜을 거슬러 올라가 그 프로젝트 소속인지로 판정하며, 아무 데도 안 붙은 엔트리는 항상 통과시킨다.

## provenance
- 라우터·핸들러: `daemon/src/routes/knowledge.rs`의 `GET /knowledge`(목록, task_id/unit_id/plan_id/project/type 필터)와 `POST /knowledge`(생성). 생성 핸들러는 embed_knowledge_atomic으로 임베딩을 동기 수행하고 실패 시 knowledge::delete로 롤백한다. 프로젝트 매칭은 plan/unit/task를 조인해 project_id 일치를 확인한다.
- 제공 컴포넌트: clawketd 데몬(component.daemon).

## 미확정 (OPEN)
- [ ] OPEN: 허용되는 지식 유형(type) 어휘 집합(decision/note/evidence/reference 등)의 권위 있는 정의 위치 확인.
