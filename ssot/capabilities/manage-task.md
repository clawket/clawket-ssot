---
id: capability.manage-task
kind: Capability
title: 태스크 생성·편집·상태 전환·삭제
purpose: "데스크탑 앱에서 태스크를 만들고 속성을 편집하며, 칸반 드래그나 상태 모달로 상태를 바꾸고(done은 evidence 필수), 삭제하거나 취소(이유는 코멘트로 보존)한다."
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/TaskEditModal.tsx
  - desktop/apps/desktop/src/shell/TaskStatusModal.tsx
  - desktop/apps/desktop/src/views/BoardView.tsx
  - desktop/apps/desktop/src/data/DataProvider.tsx
relatesTo:
  - { to: concept.task, type: mutates, note: "생성·편집·상태 전환·삭제/취소의 대상 엔티티" }
  - { to: concept.tier, type: relates-to, note: "편집 모달에서 태스크의 모델 등급(low/med/high)을 지정" }
  - { to: concept.comment, type: relates-to, note: "취소 사유가 태스크 코멘트 스레드로 보존됨" }
  - { to: concept.cycle, type: depends-on, note: "보드 칸반은 선택된 사이클에 속한 태스크를 다룬다" }
  - { to: endpoint.cli-task, type: relates-to, note: "같은 태스크 라이프사이클을 CLI에서도 제공" }
impacts: [concept.task]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 사용자가 할 수 있는 일

태스크는 사람이 직접 다루는 유일한 원자적 작업 단위다. 사람 운영자는 데스크탑 앱에서 태스크를 만들고, 제목·본문·우선순위·담당자·모델 등급(tier)·라벨 같은 속성을 편집하고, 상태를 진행시키고, 더 이상 필요 없으면 삭제하거나 취소한다.

여기에는 제품의 핵심 규칙이 녹아 있다. 태스크 상태는 정해진 전이만 허용되고, 완료(done)로 보내려면 무엇으로 끝났음을 증명하는 근거(evidence)를 반드시 입력해야 한다 — 근거 없는 완료는 데몬이 거부한다. done과 cancelled는 종착 상태라 그 뒤로는 상태 변경이 제공되지 않는다(이어서 할 일이 있으면 후속 태스크를 새로 만든다). 취소할 때 적는 사유는 파괴적으로 사라지지 않고 코멘트로 남아 히스토리가 된다.

## 행위

- 편집: 제목(필수)·본문·우선순위(low/med/high/critical)·담당자·모델 등급(low/med/high)·라벨(쉼표 구분)을 수정한다. 바뀐 값만 추려 최소 변경으로 저장하며, 본문·담당자를 비우면 비우는 의미로 처리된다.
- 상태 전환(모달): 현재 상태에서 허용된 다음 상태만 고를 수 있다 — todo→(in_progress/blocked/cancelled), in_progress→(blocked/done/cancelled), blocked→(todo/in_progress/cancelled). done 선택 시 evidence 입력이 필수이고, cancelled 선택 시 사유(선택)는 코멘트로 기록되며, 그 외 전환에는 선택적 코멘트를 붙일 수 있다.
- 상태 전환(드래그): 보드 칸반에서 카드를 다른 컬럼(todo/in_progress/blocked/done)으로 끌어다 놓으면 그 상태로 바꾼다. 5px 임계로 클릭과 드래그를 구분한다.
- 삭제/취소: 태스크를 삭제하거나(취소 사유 전달 가능) cancelled 상태로 보내 작업을 종료한다.

## 시스템 흐름

보드 칸반에서 카드를 드롭하면 데이터 계층의 태스크 수정 함수가 새 상태로 호출되고, 데몬이 전이 규칙을 검증한 뒤 반영한다. 더 세밀한 전환(특히 done의 evidence, cancelled의 사유)은 상태 모달에서 처리되며, 모달은 데몬이 evidence 누락을 거부(EVIDENCE_REQUIRED)하면 그 오류를 노출한 채 열려 재입력을 받는다.

속성 편집은 편집 모달에서 변경분만 패치로 만들어 저장하고, 변경이 없으면 저장 버튼이 비활성화된다. 모든 상태 변경·수정·삭제는 데이터 계층 `DataProvider.tsx`의 함수를 통해 데몬에 반영되고, 보드는 선택된 사이클에 속한 태스크를 상태별 컬럼으로 다시 그린다.

## 어디에 구현되어 있나

속성 편집 모달은 `TaskEditModal.tsx`, 상태 전환 모달은 `TaskStatusModal.tsx`(전이 규칙은 데몬 검증을 미러링)다. 드래그앤드롭 칸반과 카드 인터랙션은 보드 화면 `BoardView.tsx`에 있고, 생성·수정·삭제 호출은 데이터 계층 `DataProvider.tsx`에 모여 있다(삭제는 취소 사유를 옵션으로 받는다).

## 미확정 (OPEN)
- [ ] OPEN: 태스크 신규 "생성" 진입점의 정확한 UI 위치는 본 배치 소스(편집/상태 모달·보드) 밖이라 부분 확인 — 데이터 계층에는 태스크 생성 함수가 존재.
- [ ] OPEN: 데스크탑 앱 미배포(설치 게이트 no-op) 상태라 출시 빌드 동작 검증 미확인.
