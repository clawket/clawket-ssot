---
id: endpoint.projects-by-cwd
kind: Endpoint
title: 작업 디렉터리로 프로젝트 역조회
definition: "GET /projects/by-cwd/{cwd} — 주어진 작업 디렉터리 경로가 어느 프로젝트에 속하는지 역으로 찾아 돌려준다(정확 일치 우선, 없으면 가장 가까운 상위 경로로 매칭)."
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/projects.rs
  - daemon/src/repo/projects.rs
relatesTo:
  - { to: concept.project, type: reads, note: "cwd 경로로 소속 Project를 역조회" }
governedBy:
  - invariant.project-cwd-unique
governs: []
dependsOn:
  - endpoint.projects-cwds-post
impacts: []
consumedBy:
  - endpoint.cli-project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 정의
"지금 이 폴더는 어느 프로젝트인가"를 답해 주는 역조회 창구다. Claude Code 세션이 시작된 디렉터리 경로를 받아, 훅이 어느 프로젝트의 규칙·대시보드를 주입해야 할지 판정할 때 핵심으로 쓰인다. 등록된 작업 디렉터리(POST로 묶은 cwd)가 매핑의 근거다.

## 요청 / 응답
조회할 디렉터리 경로를 경로 파라미터로 넘긴다(앞에 슬래시가 없으면 절대 경로로 보정한다). 응답으로는 그 디렉터리가 속한 프로젝트 정보 전체를 돌려주며, 매칭되는 프로젝트가 없으면 찾을 수 없음으로 응답한다. 매칭은 두 단계로 일어난다 — 먼저 등록된 cwd와 정확히 같은 경로를 찾고, 없으면 그 디렉터리의 상위 경로 중 등록된 것 가운데 가장 깊은(가장 구체적인) 것을 고른다. 이렇게 하면 하위 폴더에서 작업해도 부모 프로젝트로 올바르게 귀속된다.

옵션으로 "활성 프로젝트만" 한정해 조회할 수 있다 — 켜면 훅 관리가 비활성화된 프로젝트는 매칭에서 제외된다(기본은 활성·비활성 모두 대상).

## 권한 / 제약
로컬 전용으로 인증 없이 호출한다. 읽기 전용이라 상태를 바꾸지 않는다. cwd 유일성 불변식 덕분에 한 경로의 정확 일치 매칭은 최대 하나로 확정되며, 상위 경로 매칭에서 여러 후보가 있을 때는 가장 긴(가장 구체적인) 경로가 이긴다. 이 동작은 작업 디렉터리 등록(cwds POST)이 만들어 둔 매핑에 의존한다.

## provenance
데몬 라우터의 `/projects/by-cwd/{*cwd}` GET 핸들러가 진입점이고(`daemon/src/routes/projects.rs`), 정확 일치→상위 경로 매칭 2단계 로직은 프로젝트 저장소의 역조회 구현에 있다(`daemon/src/repo/projects.rs`). 제공 주체는 데몬 컴포넌트.

## 미확정 (OPEN)
- [ ] OPEN: 심볼릭 링크·대소문자 차이·후행 슬래시 등 경로 정규화 경계 케이스의 일관성 확인 필요 — 판정은 owner.
