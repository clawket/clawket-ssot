---
id: endpoint.projects-get
kind: Endpoint
title: 프로젝트 목록 조회
definition: "GET /projects — 데몬에 등록된 모든 프로젝트(워크스페이스)를 최신 등록순으로 돌려준다."
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/projects.rs
relatesTo:
  - { to: concept.project, type: reads, note: "등록된 Project 엔티티 전체를 읽어 반환" }
governedBy: []
impacts: []
consumedBy:
  - endpoint.cli-project
  - component.web-dashboard
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 정의
지금까지 등록된 프로젝트(작업 워크스페이스)들이 무엇무엇인지 한눈에 보기 위한 조회 창구다. 웹 대시보드가 프로젝트 선택 화면을 그릴 때, CLI의 프로젝트 목록 명령이 표를 출력할 때 이 엔드포인트를 사용한다.

## 요청 / 응답
요청은 별도 입력 없이 보낸다. 응답으로는 등록된 프로젝트들의 목록을 돌려주며, 각 항목은 프로젝트를 식별하고 표시하는 데 필요한 정보 — 이름, 설명, 사람이 읽는 키, 훅 관리 활성화 여부, 위키 경로, 생성·수정 시각 — 의 의미를 담는다. 목록은 가장 최근에 만들어진 프로젝트가 위로 오도록 정렬되어 돌아온다.

## 권한 / 제약
로컬 전용으로 인증 없이 같은 머신의 클라이언트가 호출한다. 읽기 전용이라 상태를 바꾸지 않으며, 필터 없이 활성·비활성을 가리지 않고 전부 돌려준다(활성만 보기는 cwd 역조회 쪽에서 옵션으로 제공). 프로젝트가 하나도 없으면 빈 목록이 돌아온다.

## provenance
데몬 라우터의 `/projects` GET 핸들러로 구현되며(`daemon/src/routes/projects.rs`), 프로젝트 저장소의 목록 조회를 그대로 노출한다. 제공 주체는 데몬 컴포넌트.

## 미확정 (OPEN)
- [ ] OPEN: 프로젝트 수가 많아질 때의 페이징·정렬 옵션 도입 여부 — 현재는 무필터 전체 반환. 판정은 owner.
