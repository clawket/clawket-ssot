---
id: capability.manage-unit
kind: Capability
title: 유닛 생성·편집·삭제
purpose: "플랜 아래에 태스크를 묶는 그룹핑 단위인 유닛을 만들고, 제목과 목표를 다듬고, 필요 시 삭제한다."
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/UnitCreateModal.tsx
  - desktop/apps/desktop/src/shell/UnitEditModal.tsx
  - desktop/apps/desktop/src/data/DataProvider.tsx
relatesTo:
  - { to: concept.unit, type: mutates, note: "생성·편집·삭제의 대상 엔티티" }
  - { to: concept.plan, type: depends-on, note: "유닛은 반드시 한 플랜 아래에서 생성된다" }
  - { to: endpoint.cli-unit, type: relates-to, note: "같은 유닛 관리 동작을 CLI에서도 제공" }
impacts: [concept.unit]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 사용자가 할 수 있는 일

유닛은 플랜 안에서 태스크들을 묶는 순수 그룹핑 단위다. 사람 운영자는 데스크탑 앱에서 유닛을 만들어 "이 작업 묶음의 제목과 이루려는 목표"를 적어 두고, 나중에 그 묶음 아래에 사이클과 태스크를 배치한다. 유닛 자체에는 별도의 상태나 승인 절차가 없으므로, 여기서의 관리는 "묶음을 만들고, 이름·목표를 다듬고, 더 이상 필요 없으면 지우는" 단순한 형태다.

## 행위

- 생성: 제목(필수)과 목표(선택)를 입력해 한 플랜 아래에 유닛을 만든다.
- 편집: 제목과 목표를 수정한다. 바뀐 값만 추려 최소 변경으로 저장하며, 목표를 비우면 비우는 의미로 처리된다.
- 삭제: 유닛을 삭제한다.

(배치 설명에 언급된 "실행 모드" 입력란은 현재 생성·편집 모달의 실제 입력 항목에는 없다 — 두 모달은 제목과 목표만 받는다.)

## 시스템 흐름

유닛 생성은 플랜 상세 표면에서 생성 모달을 열어 시작하고, 저장하면 데이터 계층을 통해 해당 플랜 아래 유닛으로 등록된다. 편집·삭제 역시 유닛 상세 표면(상세 패널)에서 호출되어 데이터 계층의 대응 함수(유닛 생성/수정/삭제)를 거쳐 데몬에 반영된다. 편집 모달은 제목이 비었거나 변경분이 없으면 저장을 막고, 저장 실패 시 오류를 노출한 채 재시도하게 한다.

## 어디에 구현되어 있나

생성 모달은 `UnitCreateModal.tsx`(플랜 식별자 필수), 편집 모달은 `UnitEditModal.tsx`다. 실제 생성·삭제 호출 등 상태 변경은 데이터 계층 `DataProvider.tsx`에 모여 있고, 상세 패널이 이 함수들을 받아 모달과 삭제 동작을 띄운다.

## 미확정 (OPEN)
- [ ] OPEN: 배치 oneline의 "실행 모드 편집"은 코드상 모달 입력에 존재하지 않음 — 다른 표면의 별도 기능인지 또는 설명 오기인지 미확인.
- [ ] OPEN: 데스크탑 앱 미배포(설치 게이트 no-op) 상태라 출시 빌드 동작 검증 미확인.
