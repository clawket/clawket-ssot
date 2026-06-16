---
id: capability.manage-plan
kind: Capability
title: 플랜 생성·승인·완료·편집·삭제
purpose: "데스크탑 앱에서 플랜을 만들고, draft 상태를 승인해 active로 올려 태스크 시작 자격을 주며, 완료 처리하거나 제목·설명을 다듬고 필요 시 삭제한다."
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/PlanCreateModal.tsx
  - desktop/apps/desktop/src/shell/PlanEditModal.tsx
  - desktop/apps/desktop/src/shell/Sidebar.tsx
  - desktop/apps/desktop/src/data/DataProvider.tsx
relatesTo:
  - { to: concept.plan, type: mutates, note: "생성·승인·완료·편집·삭제의 대상 엔티티" }
  - { to: concept.project, type: depends-on, note: "플랜은 반드시 한 프로젝트 아래에서 생성된다" }
  - { to: endpoint.cli-plan, type: relates-to, note: "같은 플랜 라이프사이클을 CLI에서도 제공" }
impacts: [concept.plan, concept.task]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 사용자가 할 수 있는 일

플랜은 "승인된 의도를 담는 컨테이너(로드맵)"이고, 사람 운영자는 데스크탑 앱에서 그 플랜의 전체 흐름을 손으로 관리한다. 핵심 가치는 두 가지다. 첫째, 무엇을 하기로 했는지를 제목과 설명으로 명시해 등록한다. 둘째, 그 플랜을 승인(active)으로 올려 비로소 그 아래 태스크들을 시작할 수 있게 하는 게이트를 사람이 직접 통과시킨다.

이 게이트가 의도된 규칙이다 — 플랜이 active가 아니면 태스크 시작이 막히므로, 승인 행위는 "이 방향으로 일해도 좋다"는 사람의 의사결정 지점이 된다.

## 행위

- 생성: 제목(필수)과 설명, 그리고 출처(직접 작성 / Plan Mode / 가져오기 중 하나)를 지정해 새 플랜을 만든다. 플랜은 반드시 한 프로젝트에 속한다.
- 승인: draft 상태 플랜을 승인해 active로 전환한다. 이 전환이 끝나야 해당 플랜 아래 태스크를 시작할 수 있다.
- 완료: 작업이 끝난 플랜을 completed로 마무리한다. 완료된 플랜에서는 새 유닛 추가 같은 동작이 더 이상 제공되지 않는다.
- 편집: 제목과 설명을 수정한다. 바뀐 값만 추려 최소 변경으로 저장하고, 설명을 비우면 비우는 의미로 처리된다.
- 삭제: 플랜을 삭제한다.

## 시스템 흐름

새 플랜은 사이드바의 "New plan" 진입점에서 생성 모달을 열어 만든다. 저장하면 데이터 계층을 통해 활성 프로젝트 아래 플랜으로 등록된다. 승인은 사이드바의 플랜 노드에 붙은 승인 동작에서 트리거되어, 데이터 계층이 데몬에 active 전환을 요청한다.

완료·편집·삭제는 플랜 상세 표면(상세 패널/드로어)에서 호출되며, 모두 데이터 계층의 대응 함수(플랜 완료/수정/삭제)를 거쳐 데몬에 반영된다. 편집 모달은 변경분이 없으면 저장 버튼을 비활성화하고, 저장 실패 시 오류를 노출한 채 열려 있어 재시도하게 한다.

## 어디에 구현되어 있나

생성 모달은 `PlanCreateModal.tsx`(프로젝트 식별자 필수), 편집 모달은 `PlanEditModal.tsx`다. 사이드바(`Sidebar.tsx`)는 새 플랜 생성 진입점과 승인 동작을 배치한다. 생성·승인·완료·삭제 등 실제 상태 변경 호출은 데이터 계층 `DataProvider.tsx`에 모여 있고, 화면 컴포넌트는 이 함수들을 받아 사용한다.

## 미확정 (OPEN)
- [ ] OPEN: 편집·완료·삭제 동작을 사용자가 실제로 트리거하는 정확한 UI 위치(상세 패널 vs 드로어)는 본 배치 소스 밖이라 부분 확인.
- [ ] OPEN: 데스크탑 앱 미배포(설치 게이트 no-op) 상태라 출시 빌드 동작 검증 미확인.
