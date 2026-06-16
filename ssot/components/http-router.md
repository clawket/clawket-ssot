---
id: component.http-router
kind: SystemComponent
title: HTTP 라우터(요청 분배)
definition: 데몬에 들어온 모든 요청을 영역별 처리 묶음으로 갈라 주는 단일 분배 지점. 프로젝트·플랜·유닛·사이클·태스크를 비롯해 27개 서브라우터를 하나로 합쳐 데몬의 전체 기능 표면을 구성한다.
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
realizedBy: []
implementedIn:
  - daemon/src/routes/mod.rs
dependsOn:
  - component.repo-layer
consumesApi: []
providesApi:
  - endpoint.daemon-events
  - endpoint.daemon-search
integratesWith: []
impacts:
  - domain.work-management
  - domain.knowledge-rag
relatesTo:
  - to: component.clawketd
    type: belongs-to
    note: 데몬 프로세스가 부팅 시 이 라우터를 헬스 경로와 합쳐 두 리스너에 동시에 건다.
---

## 책임

HTTP 라우터는 데몬으로 들어온 요청을 "이 요청은 누가 처리해야 하는가"로 갈라 주는 단일 분배 지점이다. Clawket 의 기능이 워낙 넓어서(작업관리 워크플로우, 지식·검색, 실시간 이벤트, 검증 루프, 백업 등) 처리 로직을 한 덩어리로 두면 관리가 불가능하다. 그래서 영역별로 처리 묶음(서브라우터)을 따로 두고, 라우터는 그 묶음들을 하나의 큰 요청 표면으로 합치는 책임만 진다.

합쳐지는 영역은 27개다 — 프로젝트, 플랜, 유닛, 사이클, 태스크, 코멘트, 라벨, 관계, 지식, 런(실행 기록), 질문, 활동, 관리(admin), 감사, 백로그, 임베딩, 이벤트, 타임라인, 에이전트, 대시보드, 검증 루프(discover), 위키, 핸드오프, 가져오기·내보내기, 잠금, 사용량, 메트릭, 정적 파일. 이 합쳐진 결과가 곧 데몬이 외부에 제공하는 전체 기능 목록이다.

## 경계와 의존

라우터 자체는 데이터를 직접 만지지 않는다. 각 서브라우터가 실제 데이터 읽기·쓰기를 할 때는 그 아래 저장소 계층(`component.repo-layer`)에 위임한다. 라우터의 책임은 "분배"까지이고, "무엇을 저장하느냐"는 저장소 계층의 책임이다.

이 라우터는 데몬 프로세스(`component.clawketd`)의 일부로, 데몬이 부팅할 때 헬스 확인 경로와 함께 묶여 TCP·Unix 소켓 두 리스너에 동시에 걸린다.

## 통신 패턴

외부와는 REST 스타일 HTTP 로 통신하며, 변경 사건을 실시간으로 밀어 주는 이벤트 스트림(`endpoint.daemon-events`)과 키워드·의미·하이브리드 검색 표면(`endpoint.daemon-search`)도 이 합쳐진 표면 위에 함께 노출된다. 라우터로 합쳐진 표면 전체에 공통 정책이 한 겹 덧씌워진다 — 데이터베이스 구조 마이그레이션이 도는 동안 변경 요청을 잠시 막는 규칙이 그것이다. 이 공통 정책을 라우터 합산 결과의 바깥 레이어에 두기 때문에, TCP 로 오든 Unix 소켓으로 오든 동일하게 적용된다.

## 하위 서브패키지 (책임 단위)

서브라우터들은 Clawket 의 도메인 경계를 그대로 반영한다. 작업관리 흐름(프로젝트→플랜→유닛→사이클→태스크)에 해당하는 묶음, 지식·검색·임베딩에 해당하는 묶음, 실시간 관찰(이벤트·타임라인)에 해당하는 묶음, 검증 루프(discover) 자동화에 해당하는 묶음 등으로 나뉜다. 각 묶음의 세부 동작은 해당 영역의 Endpoint·Concept 노드에서 다룬다.

## 미확정 (OPEN)
- [ ] OPEN: 27개 서브라우터 각각이 노출하는 개별 경로 목록은 영역별 Endpoint 노드에서 별도 정리 필요.
