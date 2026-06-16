---
id: capability.manage-cycle
kind: Capability
title: 사이클 생성·활성화·완료·편집·삭제
purpose: "보드 툴바와 상세 표면에서 유닛에 묶인 타임박스 반복(사이클)을 만들고, planning에서 active로 활성화하고, 완료하고, 제목·목표를 다듬거나 삭제한다."
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/CycleCreateModal.tsx
  - desktop/apps/desktop/src/shell/CycleEditModal.tsx
  - desktop/apps/desktop/src/views/BoardView.tsx
  - desktop/apps/desktop/src/data/DataProvider.tsx
relatesTo:
  - { to: concept.cycle, type: mutates, note: "생성·활성화·완료·편집·삭제의 대상 엔티티" }
  - { to: concept.unit, type: depends-on, note: "사이클은 반드시 한 유닛에 묶인다(Cycle ⊂ Unit)" }
  - { to: concept.task, type: relates-to, note: "보드는 선택된 사이클의 태스크만 칸반으로 보여준다" }
  - { to: endpoint.cli-cycle, type: relates-to, note: "같은 사이클 라이프사이클을 CLI에서도 제공" }
impacts: [concept.cycle]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 사용자가 할 수 있는 일

사이클은 단일 유닛에 묶인 타임박스 반복(스프린트에 해당)이다. 사람 운영자는 보드 화면에서 어떤 사이클을 볼지 고르고, 새 사이클을 만들고, 사이클의 상태를 planning→active→completed로 진행시킨다. 보드는 항상 "선택된 사이클"의 태스크만 칸반으로 보여주므로, 사이클 관리가 곧 보드에서 보이는 작업 범위 관리다.

상태 진행에는 의도된 규칙이 있다. planning 사이클을 단순 필드 수정으로 active로 바꾸는 것은 허용되지 않고, 전용 활성화 동작을 거쳐야 한다(시작 시각이 서버에서 기록됨). 완료된 사이클은 다시 시작할 수 없으며, 보드의 상태 전환 컨트롤도 완료 상태에서는 잠긴다 — 같은 일을 이어가려면 새 사이클을 만든다.

## 행위

- 생성: 제목(필수)과 목표(선택)를 입력해 사이클을 만든다. 사이클은 반드시 프로젝트와 유닛에 속하며, 유닛이 비면 데몬이 생성을 거부한다.
- 활성화: planning 사이클을 active로 올린다. 보드 상태 전환에서 active를 고르면 전용 활성화 경로가 호출되어 시작 시각이 서버 측에 남는다.
- 완료: active 사이클을 completed로 마무리한다. 완료 후 재시작은 불가하다.
- 편집: 제목과 목표를 수정한다. 상태(status)는 이 편집 모달에서 다루지 않고 활성화/완료 전용 경로로만 바뀐다.
- 삭제: 사이클을 삭제한다.

## 시스템 흐름

보드 툴바는 현재 플랜에 속한 사이클 목록(완료된 것은 구분 표기)을 셀렉트로 보여주고, "New Cycle"·"Edit"·상태 배지·상태 전환 컨트롤을 제공한다. 사용자가 상태 전환에서 active를 고르면 활성화 경로가, completed를 고르면 완료 경로가 호출된다 — 이 둘은 일반 필드 수정과 다른 전용 동작이다. 생성·편집 모달은 보드 또는 상세 표면에서 열린다.

모든 상태 변경(생성·활성화·완료·수정·삭제)은 데이터 계층의 대응 함수를 거쳐 데몬에 반영되며, 보드는 데이터 계층을 통해 사이클·태스크를 읽어 다시 그린다. 상태 변경이 진행 중인 동안에는 전환 컨트롤이 잠겨 중복 요청을 막는다.

## 어디에 구현되어 있나

생성 모달은 `CycleCreateModal.tsx`(프로젝트·유닛 식별자 필수), 편집 모달은 `CycleEditModal.tsx`다. 사이클 선택·생성·편집·상태 전환의 주 표면은 보드 화면 `BoardView.tsx`이며, 활성화/완료/수정/삭제 호출은 데이터 계층 `DataProvider.tsx`에 모여 있다.

## 미확정 (OPEN)
- [ ] OPEN: 삭제 동작을 사용자가 실제 트리거하는 정확한 UI 위치(상세 드로어 vs 보드)는 본 배치 소스 밖이라 부분 확인.
- [ ] OPEN: 데스크탑 앱 미배포(설치 게이트 no-op) 상태라 출시 빌드 동작 검증 미확인.
