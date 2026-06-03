---
id: endpoint.daemon-tasks
kind: Endpoint
title: 데몬 작업 API (/tasks)
definition: "데몬이 제공하는 작업 CRUD 엔드포인트. 표준 목록·조회·생성·수정·삭제에 더해 하위 작업 생성(POST /tasks/:parent/subtasks)과 자동 분해(POST /tasks/:id/decompose)를 제공하며, 완료(done) 전환은 근거(evidence) 제출을 전제로 게이트된다."
realizedBy: [component.daemon]
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - { to: concept.task, type: mutates, note: "작업의 전체 라이프사이클(생성·수정·완료·취소)을 이 API로 다룬다" }
  - { to: concept.task-status, type: mutates, note: "todo→in_progress→done/cancelled(외부 의존 시 blocked) 상태 전이를 PATCH로 적용한다" }
  - { to: concept.task-evidence, type: reads, note: "done 전환 시 같은 요청에 근거가 포함돼야 통과한다" }
  - { to: capability.manage-task-lifecycle, type: realizes, note: "사람·에이전트의 작업 라이프사이클 관리를 이 API가 실현한다" }
  - { to: capability.decompose-task, type: realizes, note: "분해·하위작업 생성 경로를 제공한다" }
governedBy:
  - invariant.evidence-required-on-done
  - invariant.task-delete-only-draft
  - invariant.cycle-required-for-tasks
impacts: [concept.task, concept.run]
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

데몬이 작업(Task) 엔티티를 다루는 핵심 API다. 작업 목록 조회, 단건 조회, 생성, 부분 수정, 삭제 같은 표준 동작을 제공하고, 그 위에 두 가지 작업 전용 동작을 더한다 — 부모 작업 아래에 하위 작업을 만드는 경로와, 한 작업을 성공 기준에 따라 여러 하위 작업으로 자동 분해하는 경로다. 사람(웹 대시보드)과 LLM 코딩 에이전트가 모두 이 API를 통해 같은 작업 상태를 읽고 바꾼다.

작업은 워크플로우 계층(프로젝트→플랜→유닛→사이클→작업)의 가장 아래, 직접 관리하는 유일한 원자 단위다. 따라서 이 API는 일상 작업의 대부분이 통과하는 길목이다.

## 요청 / 응답

- **목록**: 어떤 범위(예: 특정 사이클)의 작업들을 돌려준다. 각 항목은 제목·상태·우선순위·담당·등급(tier)·소속(유닛/사이클)·부모 작업 등을 담는다.
- **생성**: 새 작업을 만들 때는 어느 사이클·어느 유닛에 속하는지를 반드시 함께 보내야 한다. 둘 중 하나라도 빠지면 데몬이 거부한다(사이클 누락·유닛 필요). 제목 외에 본문·우선순위·담당·등급·라벨·유형을 선택적으로 함께 보낼 수 있다.
- **수정**: 바꿀 값만 골라 보낸다. "키를 빼면 그대로 둠 / 빈 값으로 보내면 비움 / 값을 보내면 설정"의 3상태 규칙을 따른다. 상태를 완료로 바꾸는 요청에는 같은 요청 안에 근거(어디서 어떻게 충족됐는지)가 함께 들어 있어야 한다. 부모·유닛·사이클을 바꾸는 이동(re-parent/move)도 이 수정 경로로 처리되며, 데몬이 참조 무결성과 사이클 규칙 위반 여부를 검증한다.
- **삭제**: 작업이 아직 todo이고 그 플랜이 draft인 경우에는 행을 실제로 지우고, 그 외 상태에서는 삭제 대신 "취소" 상태로 전환해 히스토리를 남긴다. 취소 사유 메모를 함께 보낼 수 있다.
- **하위 작업 생성**: 부모 작업 아래에 자식을 만든다. 유닛·사이클을 생략하면 부모의 값을 물려받는다.
- **분해**: 작업의 성공 기준으로부터 하위 작업 후보를 데몬이 도출해 돌려준다. 전략·최대 깊이는 참고용 힌트이고, 실제 도출 정책은 데몬이 소유한다.

## 권한 / 제약

- 데몬은 루프백·유닉스소켓에만 바인딩되는 로컬 서비스이므로 호출 주체는 같은 머신의 사람·에이전트로 한정된다.
- 완료(done) 전환에는 근거가 필수다 — 근거 없는 완료 요청은 거부된다(governedBy: 근거 필수 불변식).
- 작업의 실제 삭제는 플랜이 draft일 때만 허용되고, 그 외에는 취소로 강등된다(governedBy: draft 한정 삭제 불변식).
- 작업 생성에는 사이클·유닛 소속이 필수다(governedBy: 사이클 필요 불변식).

## provenance

- implementedIn: 데스크탑 데몬 클라이언트가 이 엔드포인트들의 계약(경로·필수 필드·3상태 수정·완료 게이트·삭제 분기)을 주석과 함께 문서화하고 있어 데몬 측 동작을 역추론할 수 있다.
- realizedBy: clawketd 데몬이 작업 저장소(SQLite)를 소유하고 이 HTTP API를 제공한다.

## 미확정 (OPEN)
- [ ] OPEN: 분해(decompose)가 제안만 반환하는지, 수락 시 어떤 경로로 트리에 반영되는지(별도 수락 호출 여부)를 데몬 라우터에서 확정.
- [ ] OPEN: 이동(parentTaskId/unitId/cycleId 변경) 시 데몬이 검사하는 사이클·유닛 불변식의 정확한 목록.
