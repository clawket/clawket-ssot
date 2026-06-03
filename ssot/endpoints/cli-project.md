---
id: endpoint.cli-project
kind: Endpoint
title: clawket project (create/view/list/update/delete/disable/enable/resolve/cwd)
definition: 프로젝트와 그에 연결된 작업 디렉터리를 관리하고, enabled 토글로 훅 강제를 켜고 끄며, 현재 작업 디렉터리가 어느 프로젝트에 속하는지 resolve 해주는 CLI 명령군.
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.project, type: mutates, note: "프로젝트 생성·조회·수정·삭제와 작업 디렉터리(cwd) 추가/제거/목록" }
  - { to: concept.ticket-number, type: configures, note: "--key 로 티켓 번호 접두어(예: APP → APP-1)를 지정" }
governedBy: [invariant.project-cwd-unique, invariant.no-work-without-task]
impacts: [domain.work-management, domain.hook-enforcement]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket project`는 작업의 최상위 컨테이너인 프로젝트를 다루는 명령군이다. 구조화된 작업(플랜·태스크 등)을 시작하려면 먼저 작업 폴더가 어느 프로젝트에 속하는지가 정해져 있어야 하므로, 이 명령군이 그 등록과 관리를 담당한다. 하위 동작은 생성·조회·목록·수정·삭제, 훅 강제 끄기/켜기(disable/enable), 작업 디렉터리→프로젝트 resolve, 그리고 프로젝트에 딸린 작업 디렉터리 관리(cwd add/remove/list)다.

## 요청 / 응답
- 생성: 프로젝트 이름(필수)과 함께 설명, 연결할 작업 디렉터리(기본은 현재 폴더), 티켓 번호용 짧은 대문자 키를 받는다. 키를 주면 그 프로젝트의 티켓이 `<키>-1`, `<키>-2` 식으로 매겨진다.
- 조회/목록: 프로젝트 ID로 상세를 보거나 전체 프로젝트 목록을 받는다.
- 수정: 이름·설명·위키 루트 경로(지식 문서를 끌어올 폴더 목록)를 바꾼다.
- 삭제: 프로젝트와 그에 딸린 모든 데이터를 함께 지운다.
- disable/enable: 그 프로젝트에서 Clawket의 훅 강제를 끄거나 다시 켠다(아래 제약 참조).
- resolve: 지정한(또는 현재) 작업 디렉터리가 어느 프로젝트에 속하는지 프로젝트 정보 전체를 돌려준다. 일치하는 프로젝트가 없으면 "없음"을 정상 응답으로 반환한다.
- cwd add/remove/list: 한 프로젝트에 작업 디렉터리를 더하거나 빼거나 나열한다(프로젝트는 여러 폴더에 걸칠 수 있다).

## 권한 / 제약
- disable는 그 프로젝트의 훅 강제를 끈다(내부적으로 enabled 값을 끈 상태로 둔다). 끈 동안에는 변경 차단·플랜 동기화 같은 훅들이 아무 일도 하지 않아 활성 태스크 없이도 코드를 고칠 수 있고, 이미 쌓인 데이터는 그대로 보존된다. 이는 웹 대시보드의 프로젝트 설정에 있는 enabled 스위치와 같은 토글이다.
- resolve는 훅 어댑터가 "이 폴더에서 강제를 적용해야 하나"를 판단하는 데 쓰이므로, enabled 상태까지 포함해 응답한다.
- 한 작업 디렉터리는 하나의 프로젝트에만 매핑된다(작업 디렉터리 유일성). 이 때문에 resolve가 단일 프로젝트로 결정될 수 있다.
- 로컬 데몬이 떠 있어야 동작한다.

## provenance
명령 정의와 하위 동작은 CLI의 `cli/src/main.rs`(ProjectAction / ProjectCwdAction 정의와 처리)에 있다. 실제 생성·조회·수정·삭제·resolve·cwd 관리는 모두 로컬 데몬의 프로젝트 HTTP API를 호출해 처리하므로 제공 주체는 CLI 바이너리와 데몬이다.

## 미확정 (OPEN)
- [ ] OPEN: 위키 루트 경로(wiki_paths)가 지식 검색·임포트에 정확히 어떻게 쓰이는지(지식/위키 관련 노드와의 연결)는 별도 확인 필요.
