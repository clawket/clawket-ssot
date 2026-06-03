---
id: endpoint.wiki-tree
kind: Endpoint
title: 지식 위키 트리 조회 (데몬 GET /wiki/tree)
definition: 부모-자식 관계로 엮인 지식 항목을 위키 트리 형태로 정렬해 돌려주는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/knowledge.rs]
relatesTo:
  - { to: concept.knowledge, type: reads, note: "부모-자식 위키 구조로 지식 항목을 읽어 정렬한다." }
  - { to: concept.plan, type: reads, note: "지정 시 특정 플랜 범위의 위키만 모은다." }
  - { to: domain.knowledge-rag, type: belongs-to, note: "지식의 위키성 표현을 제공한다." }
governedBy: []
impacts: [capability.wiki-knowledge]
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
지식을 평평한 목록이 아니라 폴더-문서처럼 계층 구조(위키 트리)로 보여주기 위한 조회 엔드포인트다. 지식 항목은 서로 부모-자식 관계를 가질 수 있는데, 이 엔드포인트는 그 관계를 따라 항목들을 트리 순서로 정렬해 돌려준다. 웹 대시보드의 위키 화면이 좌측 트리를 그릴 때 이 결과를 쓴다.

## 요청 / 응답
요청에는 어느 항목을 트리의 뿌리로 삼을지(루트 지정, 생략 시 최상위부터), 그리고 특정 플랜 범위로 좁힐지를 선택적으로 담을 수 있다. 응답은 위키 트리 순서로 정렬된 지식 항목 목록이며, 각 항목은 자신의 부모·표시 순서 같은 위치 정보를 가져 클라이언트가 그대로 들여쓰기 트리를 그릴 수 있다.

## 권한 / 제약
로컬 데몬 클라이언트가 호출하며 읽기 전용이다. 루트나 플랜 범위를 주지 않으면 전체 위키 트리를 돌려준다.

## provenance
구현은 daemon/src/routes/knowledge.rs 의 `/wiki/tree` 핸들러에 있으며 지식 저장소의 위키 트리 조회 로직을 호출한다. 제공 주체는 로컬 데몬(component.daemon)이고, 결과를 소비하는 화면은 웹 대시보드의 위키 뷰다. "지식의 부모-자식 관계와 표시 순서를 따라 트리 순서로 정렬, 루트·플랜 범위 옵션 지원"이라는 의미만 보존하면 재구현이 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 트리 정렬에서 형제 항목 간 순서 기준(수동 인덱스 vs 생성 시각)을 명확히 확인 필요 — 판정은 owner.
