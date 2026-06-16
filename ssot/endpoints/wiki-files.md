---
id: endpoint.wiki-files
kind: Endpoint
title: 위키 파일 목록 조회 (데몬 GET /wiki/files)
definition: 작업 디렉터리의 위키 경로(기본 docs/)와 루트의 마크다운 파일을 스캔해 파일 메타 목록을 돌려주는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/wiki.rs]
relatesTo:
  - { to: concept.project, type: reads, note: "프로젝트의 위키 경로 설정을 읽어 스캔 대상을 정한다." }
  - { to: domain.knowledge-rag, type: belongs-to, note: "코드 저장소의 문서 자체를 위키 화면에 노출한다." }
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
데이터베이스에 적재된 지식이 아니라, 작업 디렉터리 안에 실제 파일로 존재하는 마크다운 문서들을 그대로 목록화하는 엔드포인트다. 웹 대시보드의 위키 화면이 "이 저장소에 어떤 문서 파일이 있는지"를 보여줄 때 쓴다. 가져오기를 거치지 않고도 코드 저장소의 docs 폴더나 루트의 README·CHANGELOG 같은 문서를 위키처럼 탐색할 수 있게 한다.

## 요청 / 응답
요청에는 스캔 기준이 되는 작업 디렉터리 경로와 대상 프로젝트를 담는다. 스캔 대상은 프로젝트에 설정된 위키 경로들(없으면 docs)이며, 거기에 더해 작업 디렉터리 루트에 바로 놓인 마크다운 파일도 함께 모은다.

응답은 발견된 마크다운 파일 목록이며, 각 항목은 상대 경로·파일명·제목·크기·마지막 수정 시각·어느 위키 루트에서 나왔는지를 담는다. 제목은 문서 앞부분의 1단계 제목 줄에서 뽑고 없으면 파일명을 쓴다. 점(.)으로 시작하는 항목과 node_modules 는 제외하고, 일정 깊이까지만 하위 폴더를 따라가며, 너무 큰 파일은 건너뛴다.

## 권한 / 제약
로컬 데몬 클라이언트가 호출하며 읽기 전용으로 파일을 읽기만 한다. 작업 디렉터리 경로가 비어 있거나 존재하지 않으면 빈 목록을 돌려준다(오류 대신 빈손). 파일 본문을 직접 열어 보는 것은 별도의 단일 파일 조회 경로가 담당하며, 그쪽은 허용된 위키 루트 밖 경로 접근을 막는다.

## provenance
구현은 daemon/src/routes/wiki.rs 의 `/wiki/files` 핸들러(디렉터리 스캔)와, 짝이 되는 단일 파일 조회 `/wiki/file` 에 있다. 제공 주체는 로컬 데몬(component.daemon)이고, 소비 화면은 웹 대시보드의 위키 뷰다. "위키 경로(없으면 docs)와 루트 마크다운을 제한 깊이로 스캔, 숨김·node_modules·대용량 제외, 첫 제목 줄에서 제목 추출, 파일 메타 반환"이라는 정책만으로 재구현이 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 데이터베이스 지식(트리)과 파일 시스템 위키 파일을 위키 화면에서 어떻게 통합·구분해 보여주는지 확인 필요 — 판정은 owner.
