---
id: endpoint.projects-cwds-post
kind: Endpoint
title: 프로젝트 작업 디렉터리 등록
definition: "POST /projects/{id}/cwds — 지정한 프로젝트에 작업 디렉터리(cwd) 경로를 묶어, 그 폴더에서의 작업이 해당 프로젝트로 인식되게 한다."
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/projects.rs
relatesTo:
  - { to: concept.project, type: mutates, note: "프로젝트에 cwd 경로를 추가해 cwd→프로젝트 매핑을 형성" }
governedBy:
  - invariant.project-cwd-unique
impacts:
  - endpoint.projects-by-cwd
consumedBy:
  - endpoint.cli-project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 정의
하나의 프로젝트가 어느 폴더(들)에서 일어나는 작업을 자기 것으로 인식할지 연결해 주는 등록 창구다. Claude Code가 특정 디렉터리에서 세션을 시작했을 때 "여기는 어느 프로젝트인가"를 데몬이 판정할 수 있으려면, 그 디렉터리 경로가 프로젝트에 미리 묶여 있어야 한다. 이 엔드포인트가 그 묶음을 추가한다. 한 프로젝트는 여러 작업 디렉터리를 가질 수 있다.

## 요청 / 응답
대상 프로젝트를 경로로 지목하고, 등록할 작업 디렉터리의 절대 경로를 본문으로 보낸다. 응답으로는 cwd 묶음이 갱신된 프로젝트 정보 전체를 돌려준다(대상 프로젝트가 없으면 찾을 수 없음으로 응답). 같은 경로를 제거하는 반대 동작(작업 디렉터리 해제)도 동일 경로의 삭제 요청으로 제공된다.

## 권한 / 제약
로컬 전용으로 인증 없이 호출한다. 하나의 cwd 경로는 한 프로젝트에만 묶이도록 유일성이 강제된다(작업 디렉터리 유일성 불변식) — 같은 폴더가 둘 이상의 프로젝트로 동시에 해석되면 cwd 역조회가 모호해지기 때문이다. 여기서 등록된 매핑은 곧바로 cwd→프로젝트 역조회 엔드포인트의 판정 근거가 된다.

## provenance
데몬 라우터의 `/projects/{id}/cwds` POST(추가)·DELETE(제거) 핸들러로 구현된다(`daemon/src/routes/projects.rs`). 제공 주체는 데몬 컴포넌트.

## 미확정 (OPEN)
- [ ] OPEN: 이미 다른 프로젝트에 묶인 cwd를 등록 시도할 때의 정확한 응답(거부 코드/덮어쓰기 여부) 확인 필요 — 판정은 owner.
