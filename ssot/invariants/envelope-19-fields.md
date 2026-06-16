---
id: invariant.envelope-19-fields
kind: Invariant
title: 봉투 폼은 정해진 19개 항목만 입력으로 노출
definition: 실행 봉투 편집 화면은 ADR-0001 이 정한 19개 표준 항목만 입력란으로 렌더링하며, 그 밖의 키는 값이 보존되어 그대로 되돌려 저장되지만 사용자 입력란으로는 드러나지 않는다.
confidence: inferred
lastVerified: 2026-06-04
lifecycle: active
owner: TBD
tags: [status:active, "type:invariant"]
governs: [concept.execution-envelope, concept.envelope]
implementedIn:
  - web/src/types.ts
  - web/src/components/EnvelopeForm.tsx
decidedBy: []
governedBy: [component.web-dashboard]
relatesTo:
  - to: concept.envelope
    type: relates-to
    note: 봉투 편집 화면(EnvelopeForm)이 노출하는 입력 항목의 범위를 19개로 고정한다.
  - to: invariant.envelope-required-fields
    type: relates-to
    note: 봉투의 필수 항목 규칙과 동일한 19개 항목 집합 위에서 동작한다.
crossesBoundary: false
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

실행 봉투를 편집하는 화면은 ADR-0001 이 표준으로 정한 19개 항목만 입력란으로 보여준다. 화면은 이 19개를 정해진 순서대로 렌더링하며, 각 항목은 그 성격에 맞는 입력 형태(짧은 텍스트, 여러 줄 텍스트, 숫자, 목록, 선택, 읽기 전용 등)로 표시된다.

봉투 데이터에 19개 외의 다른 키가 들어 있어도, 그 값은 사라지지 않는다. 편집 화면은 그런 키를 입력란으로 보여주지 않을 뿐, 저장할 때 그대로 함께 되돌려 보존한다(왕복 보존). 즉 "화면에 보이는 것"과 "저장되는 것"이 다를 수 있으며, 화면은 19개만 다루되 미지의 키를 파괴하지 않는다.

## 깨지면 무슨 일이 일어나나

만약 화면이 19개 외의 키를 입력란으로 노출하면, 표준에 없는 임의 항목을 사용자가 직접 손대게 되어 봉투의 형상이 흐트러진다. 반대로 화면이 19개만 다루면서 나머지 키를 저장 시 버려버리면, 다른 도구나 미래 스키마가 넣어둔 값이 편집 한 번에 조용히 사라진다. 19개 노출 + 나머지 왕복 보존이라는 두 약속이 함께 지켜져야, 화면은 표준 항목만 단순하게 다루면서도 봉투 전체의 무결성을 해치지 않는다.

## 코드에서 어떻게 강제되나

표준 19개 항목의 목록과 렌더링 순서는 봉투 항목 정의(`web/src/types.ts`)에 한 곳으로 모여 있고, 이는 봉투 내보내기의 항목 집합을 그대로 반영(mirror)한다. 편집 화면(`web/src/components/EnvelopeForm.tsx`)은 이 항목 목록만을 순회하여 입력란을 만들고, 항목별로 정해진 입력 형태를 매핑한다. 19개에 없는 키는 입력란 생성 루프에서 다뤄지지 않으나, 저장 시 원본 봉투 위에 덮어쓰는 방식이라 미지의 키는 값이 보존된 채 함께 되돌아간다. 19개 목록이 한 곳에 정의되어 있어 항목 집합이 화면 전체에 걸쳐 일관되게 유지된다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — ADR-0001 자체를 가리키는 결정 노드 확정 필요
