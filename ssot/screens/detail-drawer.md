---
id: screen.detail-drawer
kind: Screen
title: 상세 드로어
definition: 화면 우측에서 슬라이드로 열리는 패널로, 사이드바·뷰에서 선택한 플랜/유닛/사이클/작업의 상세를 보여주고 그 안에서 코멘트·질문·실행기록·분해 등 작업 사이드 패널을 호스팅한다.
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/DetailDrawer.tsx
  - desktop/apps/desktop/src/shell/DetailPanels.tsx
consumesApi: [endpoint.daemon-plans, endpoint.daemon-units, endpoint.daemon-cycles, endpoint.daemon-tasks, endpoint.daemon-comments, endpoint.daemon-questions, endpoint.daemon-runs]
relatesTo:
  - to: concept.selection
    type: depends-on
    note: 선택된 엔티티(종류+id)가 있을 때만 드로어가 열린다
  - to: capability.manage-task-lifecycle
    type: relates-to
    note: 작업 상세에서 갱신·완료·취소·차단 등 라이프사이클 동작을 수행
  - to: capability.decompose-task
    type: relates-to
    note: 작업 상세의 분해 패널에서 하위 작업 제안·생성
  - to: concept.comment
    type: relates-to
    note: 코멘트 패널을 호스팅
  - to: concept.question
    type: relates-to
    note: 질문 패널을 호스팅
  - to: concept.run
    type: relates-to
    note: 실행기록 패널을 호스팅
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적
사람 운영자가 트리나 뷰에서 어떤 항목을 고르면, 그 항목의 전체 맥락을 한 패널에서 깊게 들여다보고 직접 조작하는 곳이다. 플랜이면 승인·완료·수정, 유닛이면 생성·수정·삭제, 사이클이면 활성화·완료·삭제, 작업이면 수정·삭제·하위작업 생성·분해를 비롯해 코멘트 작성, 질문 등록·답변, 실행 기록 확인까지 이 드로어 안에서 처리한다.

## UI 요소 / 입력 필드
- 헤더: "Detail" 라벨과 선택 항목 부제(제목 요약), 닫기 버튼.
- 종류별 상세 본문: 선택한 항목의 종류(plan/unit/cycle/task)에 맞는 상세 패널이 렌더된다.
- 작업 사이드 패널: 작업을 선택했을 때 코멘트 목록·작성·삭제, 질문 목록·생성·답변, 실행 기록 목록, 분해 제안 패널이 함께 나타난다.
- 항목 간 이동: 상세 안에서 연결된 유닛·사이클·작업·항목을 누르면 선택이 바뀌어 같은 드로어가 해당 상세로 갱신된다.
- 너비 조절 핸들: 좌측 경계를 드래그해 폭을 조절하며 조절 값은 로컬 저장소에 기억된다.

## 표시 데이터 / 호출 API
- 표시: 선택된 엔티티의 상세 필드와, 작업이라면 그에 딸린 코멘트·질문·실행 기록.
- 호출: 플랜 승인·완료·수정, 유닛 생성·수정·삭제, 사이클 생성·수정·활성화·완료·삭제, 작업 수정·삭제·하위작업 생성·분해, 코멘트 조회·생성·삭제, 질문 조회·생성·답변, 실행 기록 조회를 각 엔티티 기능 호출로 수행한다.

## 상태 / 엣지케이스
- 닫힘 상태: 선택이 없거나 데이터가 준비되지 않았으면 드로어는 화면 밖으로 밀려나 클릭이 통하지 않는다.
- 열림 트리거: 데이터가 준비되고 선택된 항목이 실제로 해석되면 열린다.
- 닫기: 닫기 버튼 또는 Esc로 선택을 해제해 닫는다.
- 너비 한계: 최소·최대 폭으로 보정되며 저장소 사용 불가 시 기본 폭으로 동작한다.

## 미확정 (OPEN)
- [ ] OPEN: 사이클 선택 시 상세 본문에서 노출되는 동작 범위가 플랜/유닛/작업과 동일 수준인지(분해·코멘트 등 사이드 패널 적용 여부) 확인 필요.
