---
id: screen.sidebar
kind: Screen
title: 좌측 사이드바
definition: 데스크탑 워크벤치의 좌측 패널로, 브랜드 표시·프로젝트 전환·활성 컨텍스트 요약·계획 트리(Plan→Unit→Task)를 한곳에 모은다. 너비를 드래그로 조절하고 접을 수 있다.
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/Sidebar.tsx
  - desktop/apps/desktop/src/shell/ProjectSwitcher.tsx
consumesApi: [endpoint.daemon-projects, endpoint.daemon-plans, endpoint.daemon-units, endpoint.daemon-cycles]
relatesTo:
  - to: concept.plan-tree
    type: relates-to
    note: 계획 트리 영역이 Plan→Unit→Task를 표시한다
  - to: concept.active-project
    type: depends-on
    note: 활성 프로젝트가 사이드바 전체 내용의 기준
  - to: capability.switch-project
    type: relates-to
    note: 프로젝트 스위처가 활성 프로젝트 전환을 수행
  - to: capability.plan-and-approve
    type: relates-to
    note: 새 플랜 생성과 트리에서의 플랜 승인을 시작
impacts: [concept.selection]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적
사람 운영자가 작업 공간에 들어왔을 때 "지금 어떤 프로젝트의, 어떤 계획·사이클이 진행 중이고, 그 아래 어떤 작업들이 있는가"를 한눈에 잡는 진입 패널이다. 여기서 작업할 프로젝트를 고르고, 계획 트리를 펼쳐 원하는 플랜·유닛·작업으로 이동한다. 화면 폭이 좁을 때는 패널을 접어 공간을 확보하고, 평소에는 너비를 취향대로 늘리거나 줄여 쓴다.

## UI 요소 / 입력 필드
- 상단 브랜드 영역: 제품명과 버전 표기, 그리고 패널을 접는 토글 버튼.
- 프로젝트 스위처: 현재 프로젝트 이름을 보여주는 버튼을 누르면 프로젝트 목록이 열린다. 검색창에 입력하면 이름·설명·작업 디렉터리 경로를 대소문자 구분 없이 부분 일치로 걸러낸다. 비활성화된 프로젝트는 흐린 스타일로 목록 맨 아래에 모이지만 여전히 선택할 수 있다(비활성 표시는 화면용 힌트일 뿐 접근 차단이 아니다). 목록 하단에서 새 프로젝트 만들기로 들어갈 수 있다.
- 프로젝트 설정 버튼: 활성 프로젝트가 있을 때 설정 모달을 연다.
- 활성 컨텍스트 요약: 현재 활성 플랜의 제목과, 그 플랜에 속한 활성 사이클을 한 줄로 보여준다. 활성 플랜이 없으면 "활성 플랜 없음"으로 표시한다.
- 계획 트리: Plan→Unit→Task 계층을 트리로 펼쳐 보여주며, 항목을 누르면 선택되어 상세 드로어로 이어진다. 플랜 행에서는 승인 동작을 호출할 수 있다.
- 새 플랜 만들기: 플랜 영역 헤더와 빈 상태 안내에서 플랜 생성 모달을 연다.
- 너비 조절 핸들: 우측 경계를 드래그해 폭을 조절하며, 조절된 값과 접힘 여부는 브라우저 로컬 저장소에 기억된다.

## 표시 데이터 / 호출 API
- 표시: 프로젝트 목록과 활성 프로젝트, 활성 플랜·활성 사이클, 그리고 플랜 수와 계획 트리 노드.
- 호출: 프로젝트 목록·전환·생성·설정 변경(프로젝트 기능), 플랜 목록과 승인·생성(플랜 기능), 트리 구성을 위한 유닛·사이클 정보 조회. 활성 사이클은 활성 플랜에 속한 유닛 중 활성 상태인 사이클을 골라 표시한다.

## 상태 / 엣지케이스
- 접힘 상태: 폭이 최소로 줄고 브랜드 토글과 프로젝트 머리글자만 세로로 나열되며, 머리글자를 눌러 프로젝트를 바로 전환할 수 있다.
- 빈 상태: 프로젝트는 있으나 플랜이 없으면 "아직 플랜 없음" 안내와 함께 새 플랜 만들기 버튼을 보여준다. 프로젝트 자체가 없으면 스위처는 "프로젝트 없음"을 표시한다.
- 검색 결과 없음: 스위처 목록에서 "일치 항목 없음"을 표시한다.
- 너비 한계: 최소·최대 폭 안으로 강제 보정되며, 로컬 저장소를 쓸 수 없는 환경에서는 기본 폭으로 동작하고 저장은 조용히 건너뛴다.

## 미확정 (OPEN)
- [ ] OPEN: 활성 플랜/활성 사이클의 "활성" 판별이 데몬 상태와 항상 일치하는지(여러 활성 플랜이 있을 때의 표시 우선순위) 확인 필요.
