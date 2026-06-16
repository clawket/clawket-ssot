---
id: endpoint.tasks-drift
kind: Endpoint
title: 태스크 드리프트 조회 (GET /tasks/{id}/drift)
definition: 태스크 실행 봉투가 계획한 코드 기준점과 현재 저장소 상태를 비교해, 봉투가 정한 범위 안에서 실제로 얼마나 벗어났는지(드리프트 수준)를 돌려준다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.execution-envelope
    type: reads
    note: 봉투의 대상 저장소·계획 시점 커밋·범위 경계를 읽어 드리프트 판정 기준으로 삼는다
  - to: concept.task
    type: reads
    note: 대상 태스크와 그에 활성화된 봉투를 읽는다
governedBy: []
governs: []
dependsOn: []
consumesApi: []
providesApi: []
impacts:
  - domain.work-management
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
실행 봉투가 "이 작업은 이 저장소의 이 시점부터, 이 파일 범위 안에서 한다"고 계약했을 때, 실제 코드가 그 계약에서 얼마나 벗어났는지를 측정하는 조회 기능이다. 봉투에 기록된 계획 시점 커밋과 현재 저장소의 최신 커밋 사이의 변경 파일을 비교하고, 그중 봉투가 정한 작업 범위 안에 드는 변경이 몇 건인지로 드리프트 수준(없음/경미/심각)을 판정한다. 작업이 의도한 범위를 넘어 번지고 있는지, 계획과 현실이 벌어지고 있는지를 사람이나 에이전트가 점검할 때 쓰인다.

## 요청 / 응답
요청은 대상 태스크 식별자만 필요로 한다. 판정에 필요한 정보(대상 저장소, 계획 시점 커밋, 범위 경계)는 모두 그 태스크에 활성화된 실행 봉투에서 읽어 온다.

응답은 드리프트 수준, 범위 안에서 변경된 파일 목록, 전체 변경 파일 수, 그리고 비교에 사용한 두 기준점(계획 시점 커밋과 현재 최신 커밋)을 담는다. 드리프트 수준은 범위 내 변경이 없으면 "없음", 1~2건이면 "경미", 그 이상이면 "심각"으로 분류된다. 봉투에 범위 경계가 비어 있으면 변경 전체를 범위 내로 간주한다.

## 권한 / 제약
로컬 데몬 호출자가 사용한다. 존재하지 않는 태스크는 찾을 수 없음으로 응답한다. 태스크에 활성 봉투가 없거나, 봉투에 대상 저장소·계획 시점 커밋이 비어 있으면 판정 불가로 거부한다. 봉투가 가리키는 대상 저장소가 어떤 프로젝트의 작업 디렉터리로도 등록되어 있지 않으면 경로를 찾을 수 없음으로 응답한다. 현재 커밋을 읽지 못하거나 두 시점 사이 변경 비교(git diff)가 실패하면 내부 오류로 응답한다.

## provenance
구현은 데몬의 태스크 라우터(`daemon/src/routes/tasks.rs`)의 드리프트 핸들러에 있다. 활성 봉투에서 대상 저장소·계획 커밋·범위를 추출하고, 등록된 프로젝트 경로로 해석한 뒤 현재 HEAD와의 변경 파일을 비교해 범위 내 변경 수로 수준을 매긴다. 봉투/저장소 비교를 제공하는 컴포넌트는 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 경미/심각 임계값(1~2 vs 3+)이 정책으로 고정인지, 향후 조정 대상인지 owner 확인 필요.
