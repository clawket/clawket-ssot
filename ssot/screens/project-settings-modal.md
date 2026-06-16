---
id: screen.project-settings-modal
kind: Screen
title: 프로젝트 설정 패널/모달
purpose: 운영자가 프로젝트의 이름·티켓 접두사·설명·위키 경로·작업 디렉터리를 편집하고, Clawket 관리(훅 강제) 활성/비활성을 토글한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/ProjectSettingsModal.tsx
  - web/src/components/ProjectSettings.tsx
consumesApi: []
relatesTo:
  - to: concept.project
    type: relates-to
    note: 이 화면이 편집하는 1차 대상 엔티티(이름·키·설명·cwd·위키 경로·enabled).
  - to: capability.register-project
    type: realizes
    note: 작업 디렉터리(cwd) 추가·제거로 cwd↔프로젝트 바인딩을 관리.
  - to: capability.disable-project
    type: realizes
    note: Clawket 관리 토글로 훅 강제를 켜고 끄며, 끄면 제약 없이 작업하고 데이터는 보존.
  - to: concept.ticket-number
    type: reads
    note: 티켓 접두사(key)를 설정해 사람이 읽는 티켓 번호의 prefix를 정한다.
impacts:
  - concept.project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:screen
---

<!-- 본문 섹션은 x-required-sections-by-kind.Screen 와 1:1 정렬. -->

## 화면 목적

운영자가 한 프로젝트의 정체성과 동작 범위를 설정하는 화면이다. 두 가지 표면이 있다 — 사이드바에 펼쳐지는 인라인 설정 영역과, 별도로 뜨는 모달. 둘 다 같은 프로젝트 속성을 다룬다: 프로젝트 이름·설명, 티켓 번호 접두사, 어떤 디렉터리를 이 프로젝트의 작업 공간으로 볼지(cwd), 위키가 어느 폴더를 뿌리로 삼을지(위키 경로), 그리고 가장 중요하게 — 이 프로젝트에서 Clawket의 훅 강제를 작동시킬지 여부(활성/비활성 토글)다. cwd 바인딩은 에이전트가 어떤 폴더에서 작업할 때 어느 프로젝트로 인식되는지를 결정하므로, 등록·라우팅의 핵심 설정이다.

## UI 요소 / 입력 필드

**모달(ProjectSettingsModal)** — 단일 저장 흐름:
- **이름**(필수), **티켓 접두사(key)**, **설명** 입력.
- **위키 경로**(줄당 하나), **작업 디렉터리**(줄당 하나, 절대경로) 멀티라인 입력.
- **Enabled 체크박스**: "활성 프로젝트 전환기에 노출" 의미.
- 변경분이 있을 때만 Save 활성화, Cancel/Esc로 닫기. 저장 실패 시 인라인 에러.

**인라인 패널(ProjectSettings)** — 항목별 즉시 반영:
- 이름·설명: 인라인 편집(Enter 저장·Esc 취소).
- 작업 디렉터리: 줄 단위로 추가/제거(개별 호출).
- 위키 경로: 추가/제거(마지막 한 개는 제거 불가).
- **Clawket Management 토글**: 켜짐이면 "hooks enforce task registration", 꺼짐이면 "Claude works without Clawket constraints"로 설명.
- 프로젝트 ID 읽기 표시.

## 표시 데이터 / 호출 API

현재 프로젝트의 이름·키·설명·cwd 목록·위키 경로 목록·enabled 상태를 보여준다. 모달은 바뀐 필드만 추려 한 번의 프로젝트 갱신 호출로 저장하고 성공/실패 토스트를 띄운다. 인라인 패널은 이름·설명 저장, cwd 추가·제거, 위키 경로 갱신, enabled 토글을 각각 별도의 프로젝트 갱신/디렉터리 변경 호출로 즉시 반영하고, 매 변경 후 상위의 프로젝트 새로고침 콜백을 호출한다.

## 상태 / 엣지케이스

- **이름 빈 값**: 모달에서 저장 비활성(이름은 필수).
- **변경 없음**: 모달 저장 버튼 비활성.
- **저장 중**: "Saving…" 표기, 중복 제출 방지.
- **위키 경로 최소 1개**: 인라인 패널은 위키 경로가 1개뿐이면 제거 버튼을 숨겨 빈 목록을 막는다(기본값 docs로 폴백 표시).
- **빈 목록**: cwd가 없으면 "No working directories" 안내.
- **저장 실패**: 모달은 인라인 에러 + 에러 토스트.

## 미확정 (OPEN)
- [ ] OPEN: 프로젝트 갱신·cwd 추가/제거가 호출하는 데몬 endpoint 노드 매핑(현재 SSOT에 프로젝트 갱신 HTTP endpoint 노드 부재로 consumesApi 비움).
- [ ] OPEN: 모달과 인라인 패널 중 어느 것이 현재 메인 진입 표면인지(둘 다 활성 컴포넌트로 존재).
