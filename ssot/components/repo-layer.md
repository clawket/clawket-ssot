---
id: component.repo-layer
kind: SystemComponent
title: 저장소 계층(엔티티별 데이터 접근)
definition: 프로젝트·플랜·유닛·사이클·태스크·지식·런·질문·코멘트·잠금·감사 로그 등 엔티티별로 데이터베이스 읽기·쓰기를 캡슐화한 모듈 묶음. 요청 처리 묶음(라우터)이 데이터를 만질 때 거치는 단일 진입 계층이다.
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
realizedBy: []
implementedIn:
  - daemon/src/repo/mod.rs
  - daemon/src/repo/tasks.rs
  - daemon/src/repo/knowledge.rs
dependsOn:
  - component.db-store
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - domain.work-management
  - domain.knowledge-rag
relatesTo:
  - to: component.clawketd
    type: belongs-to
    note: 데몬 프로세스가 요청을 처리할 때 실제 데이터 접근을 이 계층에 위임한다.
  - to: component.http-router
    type: depends-on
    note: 각 서브라우터가 데이터를 읽고 쓸 때 이 계층을 호출한다(라우터→저장소 방향 의존).
---

## 책임

저장소 계층은 "데이터를 어떻게 저장하고 꺼내느냐"를 한곳에 모아 두는 부품이다. 요청을 받는 쪽(라우터)이 데이터베이스 세부 사항을 직접 다루게 두면, 같은 엔티티를 다루는 코드가 여기저기 흩어지고 규칙이 어긋나기 쉽다. 그래서 엔티티마다 전담 모듈을 두고, 그 엔티티에 대한 모든 읽기·쓰기를 그 모듈 하나를 통해서만 하게 한다.

전담 모듈이 있는 엔티티는 작업관리 워크플로우의 핵심(프로젝트·플랜·유닛·사이클·태스크)뿐 아니라, 지식, 런(실행 기록), 질문, 코멘트, 잠금, 감사 로그, 타임라인, 사용량, 태스크 봉투(실행 계약), 태스크 관계, 활동 로그까지 폭넓다. 각 모듈은 자기 엔티티의 데이터 의미와 무결성 규칙을 책임진다.

## 경계와 의존

이 계층은 데이터베이스 구조가 이미 최신으로 정비된 상태를 전제로 동작하므로, 부팅·마이그레이션 부품(`component.db-store`)에 의존한다. 반대 방향으로는, 요청을 분배하는 라우터(`component.http-router`)의 각 서브라우터가 데이터를 만질 때 이 계층을 호출한다 — 즉 라우터가 저장소 계층에 의존하는 구조다.

데몬 프로세스(`component.clawketd`) 전체로 보면, 저장소 계층은 "데이터 접근의 단일 통로"라는 위치를 차지한다. 비즈니스 규칙(예: 플랜이 active 가 아니면 태스크 시작 불가, done 전환에는 근거 필수 등)을 강제하는 지점도 상당수 이 계층 안에 자리한다.

## 통신 패턴

저장소 계층은 외부 네트워크와 통신하지 않는다. 위로는 라우터의 호출을 받아 처리 결과를 돌려주고, 아래로는 데이터베이스 연결을 통해 SQLite 파일을 읽고 쓴다. 동시 요청이 각자 연결을 빌려 쓰는 모델 위에서 동작하므로, 읽기는 여러 요청이 함께, 쓰기는 짧게 줄을 서서 처리된다.

## 하위 서브패키지 (책임 단위)

- **작업관리 엔티티** — 프로젝트·플랜·유닛·사이클·태스크: Clawket 워크플로우 계층의 핵심 데이터를 다룬다.
- **지식·검색 연계** — 지식 엔티티의 저장·버전 관리. 의미 검색에 쓰일 임베딩 대상도 이 데이터에서 비롯된다.
- **부수 기록 엔티티** — 런·질문·코멘트·활동 로그·타임라인·사용량: 작업의 진행·맥락·이력을 남긴다.
- **무결성·추적** — 잠금(동시 변경 충돌 방지), 감사 로그(변조 탐지 가능한 변경 추적), 태스크 봉투·관계.

## 미확정 (OPEN)
- [ ] OPEN: 각 엔티티 모듈이 강제하는 무결성 규칙의 전체 목록은 해당 Concept·Invariant 노드에서 별도 정리 필요.
