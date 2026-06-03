---
id: endpoint.knowledge-export
kind: Endpoint
title: 지식 내보내기 (데몬 POST /knowledge/export)
definition: 저장된 지식 항목을 프로젝트의 위키 경로(기본 docs/) 아래 파일로 써 내보내는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/import_export.rs]
relatesTo:
  - { to: concept.knowledge, type: reads, note: "내보낼 대상으로 지식 항목을 읽는다." }
  - { to: concept.project, type: reads, note: "프로젝트의 위키 경로 설정을 읽어 내보낼 위치를 정한다." }
  - { to: domain.knowledge-rag, type: belongs-to, note: "지식을 코드 저장소의 문서로 환원하는 통로다." }
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
Clawket 안에 쌓인 지식을 작업 디렉터리의 실제 문서 파일로 꺼내는 엔드포인트다. 데이터베이스에만 머물던 지식을 코드 저장소의 docs 폴더 같은 곳에 마크다운(또는 형식에 맞는 파일)으로 떨궈, Git 으로 함께 버전 관리하거나 사람이 평소 쓰는 에디터로 열어 볼 수 있게 한다. 위키 지식을 코드 저장소의 진실(문서)로 환원하는 방향의 통로다.

## 요청 / 응답
요청에는 기준이 되는 작업 디렉터리 경로가 필수로 들어가고, 어떤 플랜·유닛에 속한 지식만 내보낼지 좁히는 선택 조건과 대상 프로젝트를 함께 보낼 수 있다. 내보낼 폴더는 프로젝트에 설정된 위키 경로 중 첫 번째를 쓰며, 설정이 없으면 기본값 docs 폴더로 간다. 경로가 절대경로면 그대로, 상대경로면 작업 디렉터리 기준으로 해석한다.

응답으로는 실제로 써 낸 파일 개수와, 각 항목의 식별자·제목·기록된 상대 경로 목록이 온다. 파일 이름은 제목을 슬러그(영문·숫자·하이픈·한글 허용, 공백은 하이픈으로, 소문자화)로 변환해 만들고, 확장자는 지식의 저장 형식에 따라 마크다운·JSON·YAML 중 하나로 정해진다. 본문이 비어 있는 지식은 내보내지 않는다.

## 권한 / 제약
로컬 데몬 클라이언트가 호출하며 데이터베이스는 바꾸지 않지만 파일시스템에는 쓴다. 작업 디렉터리 경로가 비어 있으면 거부하고, 대상 폴더가 없으면 자동으로 만든다. 같은 제목의 지식이 여럿이면 동일한 슬러그 파일명으로 덮어쓰기가 일어날 수 있다는 점에 유의해야 한다.

## provenance
구현은 daemon/src/routes/import_export.rs 의 `/knowledge/export` 핸들러에 있으며 프로젝트 위키 경로 조회·지식 목록 조회·파일 쓰기를 수행한다. 제공 주체는 로컬 데몬(component.daemon)이다. "지식을 위키 경로(없으면 docs)로, 제목 슬러그 + 형식별 확장자로, 빈 본문은 제외하고 파일로 쓴다"는 정책만 보존하면 재구현이 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 동일 슬러그 충돌 시 덮어쓰기가 의도된 동작인지(버전 보존이 필요한지) 확인 필요 — 판정은 owner.
