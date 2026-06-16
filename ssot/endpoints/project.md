---
id: endpoint.project
kind: Endpoint
title: clawket project (프로젝트·작업 디렉터리 관리)
definition: "clawket project — 프로젝트(워크스페이스)를 만들고 보고 수정·삭제하며, 작업 디렉터리를 연결/해제하고, 훅 강제를 켜고 끄며, 현재 디렉터리에 등록된 프로젝트를 찾아주는 CLI 명령 그룹."
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.project, type: mutates, note: "프로젝트 엔티티를 생성·수정·삭제하고 enabled 토글을 변경한다" }
  - { to: concept.ticket-number, type: reads, note: "create 시 --key 로 티켓 번호 prefix(예: APP→APP-1)를 정한다" }
governedBy: [invariant.project-cwd-unique, invariant.one-active-plan, invariant.no-work-without-task]
impacts: [domain.work-management, domain.hook-enforcement]
consumedBy: [endpoint.hook-session-start, component.claude-adapter]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

`clawket project` 는 작업의 가장 바깥 그릇인 **프로젝트(워크스페이스)** 를 다루는 명령 그룹이다. 프로젝트는 하나 이상의 작업 디렉터리에 연결되며, 구조화된 작업을 시작하려면 먼저 등록되어 있어야 한다. 사람 운영자가 새 코드베이스를 Clawket 관리 아래로 들일 때, 그리고 훅 어댑터가 "지금 이 폴더가 어느 프로젝트인지" 를 판단할 때 쓴다.

제공하는 동작:
- **만들기(create)**: 이름으로 프로젝트를 만들고 현재(또는 지정) 디렉터리를 연결한다. 설명, 그리고 티켓 번호에 쓸 짧은 대문자 키(예: `APP` → `APP-1`)를 함께 줄 수 있다.
- **보기/목록(view/list)**: 단일 프로젝트 상세 또는 전체 목록을 본다.
- **수정(update)**: 이름·설명, 그리고 위키 루트 경로 목록을 바꾼다.
- **삭제(delete)**: 프로젝트와 그에 딸린 모든 데이터를 지운다.
- **끄기/켜기(disable/enable)**: 이 프로젝트에 대한 Clawket 훅 강제를 무동작으로 만들거나 다시 켠다. 웹 대시보드의 프로젝트 설정 토글과 같은 스위치다.
- **해석(resolve)**: 주어진 작업 디렉터리에 등록된 프로젝트를 찾아 반환한다. 일치하는 프로젝트가 없으면 `null` 을 주고 정상 종료한다.
- **cwd 하위명령(cwd add/remove/list)**: 한 프로젝트에 연결된 작업 디렉터리를 추가·제거·조회한다.

## 요청 / 응답

- **만들 때** 보내는 정보: 프로젝트 이름(필수), 설명, 연결할 디렉터리, 티켓 번호용 키. 응답으로 만들어진 프로젝트 식별 정보(ID와 티켓 키 등)를 돌려준다.
- **해석할 때** 보내는 정보: 조회할 작업 디렉터리(생략 시 현재 디렉터리). 응답은 그 디렉터리에 등록된 프로젝트의 전체 정보로, 특히 **훅 강제 켜짐 여부(enabled)** 를 포함한다 — 훅 어댑터는 이 값을 보고 강제를 적용할지 결정한다. 없으면 `null`.
- **목록**은 등록된 프로젝트들을 반환하며, 각 항목은 이름·연결 디렉터리·강제 상태 등 운영자가 한눈에 식별할 정보를 담는다.

(주고받는 데이터의 정확한 필드 구성은 데몬 API 시점에 결정된다 — 여기서는 의미만 기술한다.)

## 권한 / 제약

- 한 작업 디렉터리는 하나의 프로젝트에만 유일하게 매핑된다(중복 등록 불가) — `invariant.project-cwd-unique`.
- 프로젝트마다 active 플랜은 동시에 하나만 존재할 수 있다 — `invariant.one-active-plan`. (프로젝트는 그 단일성의 경계다.)
- 끄기(disable) 상태에서는 PreToolUse·UserPromptSubmit·ExitPlanMode 훅이 무동작이 되어, 활성 태스크 없이도 편집이 허용된다. 다시 켜면(enable) 강제가 복원된다. 기존 데이터는 끄더라도 보존된다.
- 변경 작업(create/update/delete 등) 자체도 활성 태스크 없이는 훅에 막힐 수 있다 — `invariant.no-work-without-task`.

## provenance

- 명령 정의·핸들러: `cli/src/main.rs`(Commands::Project, ProjectAction, ProjectCwdAction 및 그 디스패치). 실제 데이터 변경은 데몬 HTTP API로 위임된다.
- 제공 컴포넌트: `component.cli`(명령 표면) + `component.daemon`(프로젝트 영속·해석·enabled 토글).
- 코드가 사라져도 위 본문(프로젝트의 의미, cwd 매핑·enabled 토글의 정책, resolve 의 반환 의미)만으로 재구현 가능해야 한다.

## 미확정 (OPEN)

- [ ] OPEN: `update` 의 wiki_paths 갱신이 위키 검색/트리에 미치는 구체 효과(즉시 재인덱싱 여부)는 데몬 라우트 정독으로 확인 필요.
- [ ] OPEN: delete 시 연쇄 삭제 범위(플랜·태스크·지식·감사로그 보존 여부)는 데몬 측 정책 확인 필요.
