---
id: screen.web-project-settings
kind: Screen
title: 웹 대시보드 프로젝트 설정
definition: 프로젝트의 이름·설명·작업 디렉터리·위키 경로를 관리하고, Clawket Management 토글로 훅 강제를 켜고 끌 수 있는 웹 대시보드의 프로젝트 설정 패널이다.
purpose: 사람 운영자가 프로젝트의 메타데이터(이름·설명·작업 경로·위키 경로)를 손보고, Clawket 관리를 꺼서 해당 디렉터리에서 훅 제약 없이 작업하게 하거나 다시 켤 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/ProjectSettings.tsx
  - web/src/components/ProjectSettingsModal.tsx
consumesApi: []
relatesTo:
  - to: concept.project
    type: relates-to
    note: 이 화면이 다루는 대상 엔티티가 프로젝트이며 enabled 토글을 노출한다
  - to: capability.disable-project
    type: realizes
    note: Clawket Management 토글이 프로젝트 훅 비활성화 역량의 UI 표면이다
  - to: domain.hook-enforcement
    type: relates-to
    note: 토글을 끄면 훅이 그 디렉터리를 미등록처럼 다뤄 강제가 무동작이 된다
impacts:
  - concept.project
  - domain.hook-enforcement
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:screen
---

## 화면 목적
사람 운영자가 웹 대시보드에서 특정 프로젝트의 설정을 펼쳐, 프로젝트의 기본 정보와 작업 범위를 손보는 화면이다. 가장 중요한 기능은 Clawket Management 토글로, 이 프로젝트의 훅 강제를 켜고 끄는 단일 스위치다. 끄면 훅이 그 디렉터리를 등록되지 않은 것처럼 취급해 활성 태스크·플랜·사이클 같은 제약 없이 에이전트가 자유롭게 작업하고, 다시 켜면 원래의 강제 흐름으로 복귀한다. 끄더라도 기존 데이터는 그대로 보존된다.

## UI 요소 / 입력 필드
- 펼치기 헤더: "Project Settings" 섹션을 펼치고 접는다.
- 이름·설명: 인라인 편집으로 프로젝트 이름과 설명을 고친다(Enter 저장, Escape 취소).
- 작업 디렉터리(Working Directories): 프로젝트에 연결된 cwd 목록을 보고, 새 경로를 추가하거나 기존 경로를 제거한다. cwd는 어느 디렉터리가 이 프로젝트로 인식되는지를 결정한다.
- 위키 경로(Wiki Paths): 위키 트리의 루트가 되는 경로를 추가·제거한다. 프로젝트 cwd 기준 상대 경로 또는 절대 경로이며, 최소 한 개는 유지된다(기본 docs).
- Clawket Management 토글: 켜짐이면 "훅이 태스크 등록을 강제", 꺼짐이면 "Clawket 제약 없이 Claude가 작업"으로 상태 문구가 바뀐다.
- 프로젝트 ID: 읽기 전용으로 표시된다.

## 표시 데이터 / 호출 API
보여 주는 정보는 대상 프로젝트의 이름·설명·작업 디렉터리 목록·위키 경로 목록·enabled 여부·프로젝트 ID다. 모든 편집은 데몬의 프로젝트 갱신 기능을 통해 저장되며(이름·설명·위키 경로 변경, cwd 추가/제거, enabled 토글), 저장 후 화면이 최신 프로젝트 상태를 다시 불러와 반영한다. enabled 값을 1/0으로 토글하는 것이 곧 훅 강제의 on/off 스위치다.

## 상태 / 엣지케이스
- 빈 이름은 저장하지 않는다(이름 비우기 방지).
- 작업 디렉터리가 하나도 없으면 "작업 디렉터리 없음"을 안내로 표시한다.
- 위키 경로는 항목이 둘 이상일 때만 개별 제거 버튼이 나타나, 마지막 한 개는 지워지지 않게 한다.
- Clawket Management를 꺼도 데이터는 유지되며 언제든 다시 켤 수 있다(되돌릴 수 있는 비파괴 토글).

## 미확정 (OPEN)
- [ ] OPEN: 이 패널이 호출하는 프로젝트 갱신 endpoint(데몬 HTTP/CLI project update·addCwd·removeCwd 대응)를 endpoint 노드와 정확히 연결할 것
- [ ] OPEN: 인라인 ProjectSettings 패널과 ProjectSettingsModal 의 사용 맥락 차이(어디서 어느 쪽이 뜨는지)를 코드에서 확정할 것
