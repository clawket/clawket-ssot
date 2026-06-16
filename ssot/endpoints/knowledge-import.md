---
id: endpoint.knowledge-import
kind: Endpoint
title: 지식 가져오기 (데몬 POST /knowledge/import)
definition: 작업 디렉터리의 위키 경로(기본 docs/) 아래 마크다운 문서를 훑어 지식 항목으로 적재하는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/import_export.rs]
relatesTo:
  - { to: concept.knowledge, type: mutates, note: "스캔한 문서를 지식 항목으로 생성한다." }
  - { to: concept.project, type: reads, note: "프로젝트의 위키 경로 설정을 읽어 스캔 대상을 정한다." }
  - { to: concept.plan, type: reads, note: "지정 시 가져온 지식을 해당 플랜에 첨부한다." }
  - { to: concept.unit, type: reads, note: "지정 시 가져온 지식을 해당 유닛에 첨부한다." }
  - { to: domain.knowledge-rag, type: belongs-to, note: "기존 문서를 RAG 지식으로 끌어오는 통로다." }
governedBy: []
impacts: [capability.wiki-knowledge]
consumedBy: [endpoint.cli-knowledge]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
이미 코드 저장소에 흩어져 있는 마크다운 문서를 Clawket 지식으로 한 번에 끌어오는 엔드포인트다. docs 폴더 같은 위키 경로를 훑어 각 문서를 지식 항목으로 만들어, 이후 검색·임베딩·MCP 노출의 대상이 되게 한다. 내보내기와 짝을 이루어, 문서(코드 저장소)와 지식(Clawket) 사이를 오가게 한다. 실제 적재 전에 무엇이 들어올지 미리 보는 시뮬레이션(dry-run)도 지원한다.

## 요청 / 응답
요청에는 기준 작업 디렉터리 경로가 필수이고, 가져온 지식을 어떤 플랜·유닛에 붙일지, 어떤 프로젝트의 위키 경로를 쓸지, 그리고 실제 적재 없이 미리보기만 할지(dry-run) 여부를 함께 보낼 수 있다. 스캔 대상은 프로젝트에 설정된 위키 경로들이며 설정이 없으면 docs 폴더다. 점(.)으로 시작하는 항목과 node_modules 는 건너뛰고, 일정 깊이까지만 하위 폴더를 따라가며, 너무 큰 파일은 제외한다.

응답으로는 적재된 개수·건너뛴 개수와 함께, 적재된 항목 목록(식별자·상대 경로·제목)과 건너뛴 항목 목록(경로·제목·사유)이 온다. 제목은 문서 첫 번째 1단계 제목 줄에서 뽑고 없으면 파일명을 쓴다. 같은 제목이 이미 존재하면 중복으로 보고 건너뛴다. 미리보기 모드에서는 식별자 없이 어떤 항목이 들어올지만 돌려준다.

## 권한 / 제약
로컬 데몬 클라이언트가 호출하며 미리보기가 아니면 지식을 새로 만든다(쓰기). 작업 디렉터리가 비어 있거나 존재하지 않으면 거부한다. 중복 판정은 같은 플랜·유닛 범위 안에서 이미 가진 제목 집합과 대조해 이뤄진다. 마크다운·MDX 만 대상으로 하며 형식은 파일 확장자에 따라 보존된다.

## provenance
구현은 daemon/src/routes/import_export.rs 의 `/knowledge/import` 핸들러와 디렉터리 스캔 로직에 있다. 제공 주체는 로컬 데몬(component.daemon)이다. "위키 경로(없으면 docs)를 제한 깊이로 스캔, 숨김·node_modules·대용량 제외, 첫 제목 줄에서 제목 추출, 동일 제목 중복은 스킵, 미리보기 지원, 플랜/유닛 첨부"라는 정책만으로 재구현이 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 중복 판정이 제목 동일성에만 기대는데, 제목이 같고 내용이 다른 문서의 갱신 시나리오를 어떻게 처리할지 확인 필요 — 판정은 owner.
