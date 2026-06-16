---
id: endpoint.tasks-subtasks
kind: Endpoint
title: 데몬 POST /tasks/{id}/subtasks
definition: 한 태스크 아래에 실제 하위 태스크를 만드는 통로. 부모의 소속(유닛·사이클)과 실행 계약을 물려받아 자식 태스크를 트리로 생성한다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: mutates
    note: 지정한 부모 아래에 새 하위 태스크를 생성한다
  - to: concept.task-envelope
    type: mutates
    note: 부모의 실행 봉투를 상속·오버라이드해 자식 봉투를 한 거래로 서명한다
  - to: concept.execution-envelope
    type: reads
    note: 부모의 실행 봉투를 상속 기준으로 읽는다
  - to: component.daemon
    type: backed-by
    note: 하위 태스크 생성과 봉투 서명을 한 트랜잭션으로 처리하는 로컬 데몬
governedBy:
  - invariant.secret-never-in-envelope
impacts:
  - concept.task
  - capability.decompose-task
consumedBy:
  - capability.decompose-task
  - endpoint.cli-task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
한 태스크 아래에 실제 하위 태스크를 만드는 통로다. 분해 제안 통로가 "어떻게 나누면 좋을지"를 제시만 한다면, 이쪽은 그 결정을 받아 실제로 자식 태스크를 트리에 심는 단계다. 새 자식은 부모의 소속(어느 유닛·어느 사이클에 속하는지)과 실행 계약(실행 봉투)을 기본으로 물려받아, 부모와 일관된 맥락 안에서 만들어진다. 큰 작업을 손에 잡히는 작은 단위로 쪼개 트리 구조로 관리하려 할 때 쓴다.

## 요청 / 응답
보내는 정보는 새 자식 태스크를 정의하는 항목들이다 — 제목(필수)과, 선택적으로 본문·담당자·정렬 순서·의존 관계·우선순위·복잡도·예상 편집량·소속 사이클/유닛·보고자·유형, 그리고 부모에게서 물려받은 실행 봉투에 덧씌울 부분(봉투 오버라이드)이다. 소속 유닛은 생략하면 부모와 같은 유닛, 사이클도 생략하면 부모의 사이클을 따른다. 응답으로는 새로 만들어진 자식 태스크를 그 실행 봉투와 함께 돌려준다. 부모 태스크가 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 호출된다. 핵심 제약은 "자식 생성"과 "물려받은 실행 봉투 서명"이 하나의 거래로 묶인다는 점이다 — 봉투 서명이 실패하면 자식 태스크 생성 자체가 함께 취소되어, 봉투 없는 떠돌이 자식이 남지 않는다(원자성). 봉투에는 비밀값(시크릿·고엔트로피 문자열)이 들어가서는 안 되며, 부모 봉투 상속·오버라이드 과정에서도 이 금지가 유지된다.

## provenance
근거는 `daemon/src/routes/tasks.rs`의 라우트 등록(`/tasks/{id}/subtasks`, POST → `create_subtask`)과 핸들러 `create_subtask`다. 핸들러는 부모를 조회해 유닛/사이클을 기본 상속하고(`unit_id`/`cycle_id` 생략 시 부모 값), 부모의 활성 봉투(`task_envelopes::active_for_task`)와 요청의 `envelope_overrides`로 자식 봉투를 계산(`compute_inherited_envelope`)한다. 자식 INSERT(`tasks::create_in_tx`)와 봉투 서명(`env_sign::sign_envelope_in_tx`)은 단일 트랜잭션으로 커밋되어(주석 D2 atomicity), 서명 실패 시 자식이 롤백된다. 생성 후 `task:created` 이벤트 방출과 임베딩 스케줄이 뒤따른다. 처리 주체는 로컬 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 부모 봉투의 어떤 필드가 상속되고 어떤 필드가 오버라이드로만 채워지는지(`compute_inherited_envelope`의 상속 규칙)와, 비밀값 거부 검사가 생성 경로에도 동일하게 걸리는지는 봉투 상속·서명 코드를 정독해 확인 필요.
