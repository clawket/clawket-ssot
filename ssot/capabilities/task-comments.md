---
id: capability.task-comments
kind: Capability
title: 작업 코멘트 달기·지우기
definition: 데스크탑 앱의 작업 상세 화면에서 그 작업에 코멘트를 달고, 오래된 코멘트를 지운다. 삭제는 줄 자체를 없애지 않고 "삭제됨" 표시를 남기는 부드러운 삭제이며, 코멘트는 오직 작업에만 달 수 있다.
purpose: 작업 진행 중의 메모·맥락·결정 흔적을 사람이 작업 화면에서 바로 남기고, 지운 흔적도 기록으로 보존되게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.desktop-app
implementedIn:
  - desktop/apps/desktop/src/shell/TaskCommentsPanel.tsx
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.comment
    type: relates-to
    note: 작업에 붙는 코멘트를 생성·조회·소프트삭제한다
  - to: concept.task
    type: relates-to
    note: 코멘트는 작업에만 달 수 있다
  - to: endpoint.cli-comment
    type: relates-to
    note: 같은 코멘트 데이터를 CLI 코멘트 명령과 공유한다
impacts:
  - concept.comment
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. 코드 옮겨적기 금지. -->

## 사용자가 할 수 있는 일

사람이 작업 화면을 떠나지 않고 그 작업에 코멘트를 남길 수 있다. 진행 메모, 맥락 설명, 누가 무엇을 결정했는지 같은 흔적을 작업 옆에 쌓아 둔다. 더 이상 필요 없는 코멘트는 지울 수 있는데, 이때 줄이 완전히 사라지는 게 아니라 "삭제됨"으로 표시되어 회색으로 흐려진 채 남는다 — 파괴적 삭제 대신 흔적을 보존하는 방식이다. 코멘트는 작업에만 달리며, 묶음이나 플랜에는 이 화면에서 달 수 없다.

## 행위

- 작업 상세를 열면 그 작업의 코멘트를 시간순(오래된 것부터)으로 불러와 보여준다.
- 새 코멘트를 작성해 등록한다. 작성자는 기본값("main")으로 채워져 CLI와 같은 기본 작성자를 따른다.
- 코멘트를 부드럽게 삭제한다 — 확인을 거친 뒤 지우면 본문 앞에 삭제 표시가 붙은 채 남고, 화면에서는 그 표시를 떼어 흐리게 보여주며 "삭제됨" 배지를 단다. 이미 삭제된 코멘트에는 삭제 버튼이 보이지 않는다.
- 코멘트를 추가·삭제할 때마다 목록을 다시 불러와 최신 상태를 반영한다.

## 시스템 흐름

작업 상세 → 코멘트 패널이 그 작업의 코멘트 목록을 읽어 시간순으로 렌더 → 사람이 코멘트를 작성하면 데몬에 등록 후 목록 재조회 → 삭제 시 확인 → 데몬이 본문을 삭제 표시로 바꾸는 소프트 삭제 수행 → 재조회로 삭제 표시가 화면에 반영. 코멘트 수명주기에 대한 별도 실시간 이벤트가 없어, 이 패널이 변경 때마다 스스로 다시 읽어 동기화한다.

## 어디에 구현되어 있나

데스크탑 앱의 작업 코멘트 패널(`TaskCommentsPanel.tsx`)이 목록 표시·작성·소프트삭제 UI와 재조회 흐름을 담당한다. 읽기·쓰기·삭제 호출은 데스크탑 데이터 계층(`data/api.ts`)을 거쳐 데몬으로 간다.

## 미확정 (OPEN)
- [ ] OPEN: 소프트삭제된 코멘트를 영구 삭제하거나 복구하는 경로가 있는지 미확인.
- [ ] OPEN: 코멘트 수정(편집)이 데스크탑 화면에서 지원되는지 미확인(이 패널은 작성·삭제만 노출).
