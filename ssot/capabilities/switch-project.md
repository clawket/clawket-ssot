---
id: capability.switch-project
kind: Capability
title: 프로젝트 전환·생성·설정
purpose: "데스크탑 앱 사이드바에서 작업 대상 프로젝트를 즉시 바꾸고, 새 프로젝트를 만들고, 기존 프로젝트의 메타정보(이름·티켓 접두어·설명·위키 경로·작업 디렉터리·활성 토글)를 손본다."
servesPersona: [persona.human-operator]
realizedBy: [component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/shell/ProjectSwitcher.tsx
  - desktop/apps/desktop/src/shell/ProjectCreateModal.tsx
  - desktop/apps/desktop/src/shell/ProjectSettingsModal.tsx
  - desktop/apps/desktop/src/shell/Sidebar.tsx
relatesTo:
  - { to: concept.project, type: mutates, note: "활성 프로젝트 선택·생성·메타 수정의 대상 엔티티" }
  - { to: endpoint.cli-project, type: relates-to, note: "같은 프로젝트 생성/수정 동작을 CLI에서도 제공" }
impacts: [concept.project]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 사용자가 할 수 있는 일

사람 운영자는 데스크탑 앱의 사이드바 상단에 있는 프로젝트 선택기를 눌러, 지금 보고 작업할 프로젝트를 고른다. 한 번에 하나만 활성화되며, 활성 프로젝트가 곧 보드·백로그·위키 등 모든 화면이 보여주는 범위를 결정한다.

선택기는 단순 전환을 넘어 프로젝트의 생애주기 진입점 역할을 한다. 목록 아래쪽에서 새 프로젝트를 바로 만들 수 있고, 현재 활성 프로젝트가 있으면 그 설정을 여는 항목이 함께 나타난다. 즉 "어디서 일할지 고르기 → 없으면 새로 만들기 → 골라둔 곳을 다듬기"가 한 표면에서 끝난다.

목록은 활성 프로젝트를 맨 위에, 그다음 사용 중인 프로젝트를 이름순으로, 비활성화된 프로젝트를 흐린 스타일로 맨 아래에 배치한다. 비활성 프로젝트도 여전히 선택은 가능하다 — 활성/비활성 구분은 화면 노출 우선순위를 위한 힌트이지, 접근을 막는 잠금이 아니다. 검색창에 입력하면 이름·설명·작업 디렉터리 경로를 대소문자 무시로 부분 일치 필터링한다.

## 행위

- 프로젝트 목록 열기·검색·선택(전환): 선택 즉시 활성 프로젝트가 바뀌고 메뉴가 닫힌다.
- 새 프로젝트 생성: 이름(필수)과 함께 티켓 접두어(키), 설명, 위키 경로 목록, 작업 디렉터리 목록을 입력해 만든다. 생성된 프로젝트는 곧바로 활성화된다.
- 프로젝트 설정 수정: 이름·티켓 접두어·설명·위키 경로·작업 디렉터리, 그리고 "활성(enabled)" 토글을 편집한다. 바뀐 필드만 골라 최소 변경으로 저장하며, 빈 값으로 비우면 해당 항목을 지우는 의미로 처리된다.

위키 경로와 작업 디렉터리는 한 줄에 하나씩 적는 다중 입력으로 받고, 작업 디렉터리는 절대경로로 적는다.

## 시스템 흐름

사용자가 프로젝트 선택기 버튼을 누르면 데스크탑 앱이 보유한 프로젝트 목록을 정렬·필터해 드롭다운으로 보여준다. 항목을 고르면 활성 프로젝트 식별자가 갱신되고 화면 전체가 그 프로젝트 범위로 다시 그려진다.

새 프로젝트 만들기를 고르면 생성 모달이 열리고, 저장 시 데이터 계층을 거쳐 프로젝트가 등록된 뒤 자동으로 활성화된다. 설정 항목을 고르면 활성 프로젝트로 채워진 설정 모달이 열리고, 변경분만 추려 저장한다. 저장 중 오류가 나면 모달이 열린 채 오류 메시지를 노출해 재시도하게 한다.

## 어디에 구현되어 있나

프로젝트 선택기 본체는 `ProjectSwitcher.tsx`로, 사이드바 헤더에 앵커링된 클릭형 단일 선택 목록이다. 생성 모달은 `ProjectCreateModal.tsx`, 설정 모달은 `ProjectSettingsModal.tsx`가 담당하며, 둘 다 선택기가 필요 시 띄운다. 이 선택기를 사이드바에 배치하고 활성 프로젝트 상태를 연결하는 곳은 `Sidebar.tsx`다.

## 미확정 (OPEN)
- [ ] OPEN: 데스크탑 앱 자체가 현재 정식 배포되지 않는 상태(설치 게이트 no-op)라, 이 화면 동작이 실제 출시 빌드에서 검증된 것인지는 미확인.
