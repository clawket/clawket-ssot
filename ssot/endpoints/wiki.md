---
id: endpoint.wiki
kind: Endpoint
title: 위키 대시보드 페이로드 조회
definition: "GET /dashboard/wiki — 프로젝트에 쌓인 지식 항목(결정·노트·근거·참고자료 등)을 위키 화면에 보여주기 위한 데이터를 데몬에서 가져온다."
realizedBy:
  - component.daemon
implementedIn:
  - cli/src/main.rs
  - daemon/src/routes/wiki.rs
relatesTo:
  - to: concept.knowledge
    type: lists
    note: 프로젝트에 첨부된 지식 항목들을 위키 화면용으로 묶어 나열한다.
governedBy: []
impacts: []
consumedBy:
  - component.web-dashboard
  - endpoint.cli-web-views
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

위키 뷰는 프로젝트에 쌓인 지식 항목을 문서 묶음처럼 펼쳐 보여주는 화면이다. 작업 중 내린 결정, 메모, 검증 근거, 참고 자료 같은 항목들이 여기에 모인다. 사람 운영자가 "이 프로젝트에서 그동안 무슨 결정과 기록이 쌓였는지"를 훑어보거나 특정 문서를 찾아 들어가는 출발점이 되는 화면이다. 이 노드는 그 위키 화면을 채우기 위한 데이터를 데몬에서 받아오는 조회 경로를 가리킨다.

조회 단위는 하나의 프로젝트이며, 프로젝트를 지정하지 않으면 현재 작업 디렉터리에 연결된 프로젝트를 대상으로 한다.

## 요청 / 응답

요청에는 어느 프로젝트의 위키를 볼지(생략 시 현재 디렉터리 프로젝트)를 보낸다.

응답은 지식 항목들의 목록 — 화면에 따라 파일 트리 형태로도 표현될 수 있는 구조 — 이다. 각 항목은 제목·종류(결정/노트/근거/참고 등)·소속 위치 같은, 목록에서 한 줄로 보여줄 메타 정보를 담는다. 항목 본문 전체는 보통 사용자가 특정 항목을 열 때 별도의 상세 조회로 받아온다.

## 권한 / 제약

로컬 데몬에 연결 가능한 같은 기기의 사용자·에이전트가 조회할 수 있는 읽기 전용 경로다. 지식 항목을 새로 만들거나 고치는 행위는 이 조회 경로가 아니라 별도의 지식 변경 경로에서 일어난다.

## provenance

CLI 쪽 진입점은 `cli/src/main.rs`의 위키 보기 명령으로, 데몬에 위키 대시보드 데이터를 요청해 출력한다. 데몬 쪽에서 위키 파일 트리·개별 파일을 돌려주는 조회 로직은 `daemon/src/routes/wiki.rs`에 구현되어 있다. 데이터를 소유하는 주체는 데몬(component.daemon)이며, 위키 렌더링은 웹 대시보드(component.web-dashboard)가 맡는다.

## 미확정 (OPEN)
- [ ] OPEN: CLI가 호출하는 `/dashboard/wiki` 경로에 대응하는 데몬 라우트가 현재 등록되어 있지 않다(데몬 위키 라우트는 `/wiki/files`·`/wiki/file`로 노출됨). CLI의 위키 단축 명령이 데이터를 직접 받는지 웹 뷰만 여는지, 그리고 대시보드 위키 페이로드를 만드는 핸들러의 정확한 경로를 owner 확인 필요.
- [ ] OPEN: 위키 목록 항목이 담는 메타 필드의 의미(종류·버전·첨부 대상 등)와 파일 트리 표현 규칙은 코드 정독으로 보강 필요.
