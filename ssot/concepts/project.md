---
id: concept.project
kind: Concept
title: Project
definition: "하나 이상의 작업 디렉터리(cwd)에 바인딩된 워크스페이스. 구조화된 작업을 시작하기 전 반드시 등록되어야 하며, enabled 토글로 Clawket 훅 강제를 켜고 끌 수 있다. 동의어: 워크스페이스."
relatesTo:
  - { to: concept.plan, type: contains, note: "한 프로젝트는 여러 플랜을 담는다. 프로젝트당 active 플랜은 하나만 허용된다." }
  - { to: domain.work-management, type: belongs-to, note: "작업 관리 계층의 최상위 컨테이너." }
  - { to: capability.register-project, type: backed-by, note: "작업 디렉터리를 프로젝트로 등록하는 행위로 실체화된다." }
  - { to: capability.disable-project, type: backed-by, note: "enabled 토글을 끄는 행위로 훅 강제가 무동작이 된다." }
  - { to: endpoint.cli-project, type: mutates, note: "create/update/delete/disable/enable/resolve/cwd 서브커맨드로 관리된다." }
governedBy:
  - invariant.project-cwd-unique
  - invariant.one-active-plan-per-project
  - invariant.tasks-not-project-scoped
governs: []
dependsOn: []
implementedIn:
  - daemon/src/models.rs
  - daemon/src/repo/projects.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

Project(프로젝트)는 Clawket이 작업을 추적하는 가장 바깥 단위다. 사람이 보기엔 "이 코드베이스/저장소에서 하는 일 전체를 담는 작업 공간"에 해당한다. 프로젝트는 하나 이상의 **작업 디렉터리(cwd)** 에 묶여 있어서, LLM 에이전트나 사람이 어떤 폴더에서 작업하든 그 폴더가 어느 프로젝트에 속하는지 역으로 찾아낼 수 있다(cwd → 프로젝트 resolve).

Clawket의 핵심 원칙은 "구조화된 작업이 시작되기 전에 작업 디렉터리를 프로젝트로 등록한다"이다. 등록되지 않은 폴더에서 작업을 시도하면 세션 시작 훅이 이를 감지해 사용자에게 프로젝트 등록을 권한다.

프로젝트는 그 아래에 여러 **플랜(Plan)** 을 담는 컨테이너이며, 사람이 읽기 쉬운 짧은 식별자(`key`, 예: 티켓 prefix)와 위키 경로 목록을 가진다.

## 엔티티 (DB)

프로젝트 한 건은 다음 의미의 데이터를 보존한다.

- **이름·설명·키**: 사람이 식별하는 이름, 부가 설명, 티켓 번호의 prefix로 쓰일 수 있는 짧은 키.
- **enabled(켜짐 여부)**: Clawket 관리(훅 강제)를 켜고 끄는 토글. 끄면 해당 프로젝트의 작업 디렉터리에서 훅이 무동작이 되어 제약 없이 작업할 수 있고, 이미 쌓인 데이터는 그대로 보존된다.
- **작업 디렉터리 목록(cwds)**: 이 프로젝트에 바인딩된 한 개 이상의 폴더 경로. 같은 cwd가 두 프로젝트에 동시에 묶일 수 없다.
- **위키 경로 목록(wiki_paths)**: 지식 항목을 가져오고 내보낼 문서 폴더.
- 생성·수정 시각.

태스크는 프로젝트에 직접 매달리지 않는다. 태스크는 유닛(과 그를 통해 플랜)에 속하며, 프로젝트 소속은 그 상위 체인을 거쳐 간접적으로 결정된다.

## API 표면

프로젝트는 `clawket project` 명령군으로 생성·조회·수정·삭제하고, `disable`/`enable`로 훅 강제를 토글하며, `resolve`/`cwd`로 현재 작업 디렉터리가 속한 프로젝트를 찾는다. 세션 시작 시 대시보드 명령이 활성 프로젝트의 작업 요약을 시드로 사용한다.

## 불변식

- **작업 디렉터리 유일성**: 하나의 cwd는 최대 한 프로젝트에만 바인딩된다(`invariant.project-cwd-unique`).
- **프로젝트당 active 플랜 하나**: 한 프로젝트 안에서 동시에 active일 수 있는 플랜은 하나뿐이다(`invariant.one-active-plan-per-project`).
- **태스크는 프로젝트 직속이 아님**: 태스크의 소속은 유닛→플랜→프로젝트 체인으로 간접 결정된다(`invariant.tasks-not-project-scoped`).

## 구현 위치 (provenance)

프로젝트 엔티티의 형상은 데몬 모델(`daemon/src/models.rs`)에 정의되어 있고, 생성·조회·cwd 바인딩 등 영속 로직은 `daemon/src/repo/projects.rs`에 있다.

## 미확정 (OPEN)
- [ ] OPEN: `key`가 티켓 번호 prefix로 쓰이는 규칙의 정확한 동작(프로젝트별 prefix 커스터마이즈 가능 여부) 검증 필요.
