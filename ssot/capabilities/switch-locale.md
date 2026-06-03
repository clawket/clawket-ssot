---
id: capability.switch-locale
kind: Capability
title: 언어 전환(로케일 선택)
definition: 랜딩 사이트 방문자가 헤더의 언어 드롭다운에서 지원 언어를 고르면 화면 카피가 즉시 해당 언어로 바뀌고, 문서의 언어·쓰기 방향 속성이 함께 갱신되며, 선택은 브라우저에 기억되어 다음 방문에도 유지된다.
purpose: 전 세계 방문자가 자신의 언어로 제품 소개를 읽도록 해 첫인상의 진입 장벽을 낮춘다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.landing-site
  - component.i18n
implementedIn:
  - landing/src/components/LocaleSwitcher.tsx
  - landing/src/i18n/context.tsx
  - landing/src/i18n/locales.ts
  - landing/src/i18n/dict.ts
relatesTo:
  - to: concept.locale
    type: relates-to
    note: 이 기능이 다루는 핵심 개념이 로케일이다
  - to: integration.i18n-locale
    type: relates-to
    note: 훅·설치 메시지 쪽 로케일 체인과 같은 다국어 정책 계열이나 랜딩은 별도 사전을 쓴다
impacts:
  - concept.locale
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

랜딩 사이트를 방문한 사람은 페이지 어디서든(홈, 문서 영역) 헤더 우측의 지구본 아이콘 옆 드롭다운을 열어 자신이 읽을 언어를 직접 고를 수 있다. 고르는 즉시 화면의 모든 소개 문구가 그 언어로 바뀐다. 한 번 고른 언어는 브라우저에 저장되므로, 다시 방문해도 같은 언어로 보인다. 처음 방문하는 사람은 따로 고르지 않아도 브라우저의 언어 설정에 맞춰 자동으로 가장 가까운 지원 언어가 선택된다.

지원 언어는 영어·한국어·일본어·중국어(간체/번체)·스페인어·프랑스어·독일어·포르투갈어(브라질)·러시아어·이탈리아어·아랍어·힌디어·인도네시아어·터키어·베트남어·태국어·폴란드어·네덜란드어·스웨덴어로, 모두 20개다. 이 중 아랍어를 고르면 글이 오른쪽에서 왼쪽으로 흐르도록 화면 방향까지 함께 바뀐다.

## 행위

- 읽기: 현재 적용된 언어와 지원 언어 목록을 화면에 표시한다.
- 수정(전환): 방문자가 드롭다운에서 다른 언어를 선택하면 화면 카피를 그 언어로 교체한다.
- 자동 결정: 저장된 선택이나 주소의 `lang` 파라미터가 없으면 브라우저 언어 설정에서 가장 가까운 지원 언어를 추론한다(예: 중국어 변형은 간체/번체로 정규화, 포르투갈어는 브라질 표기로 정규화).
- 저장: 선택한 언어를 브라우저 로컬 저장소에 보관해 다음 방문에 재사용한다.

이 기능은 데몬 API나 CLI를 호출하지 않는다. 모든 처리가 방문자의 브라우저 안에서만 일어난다.

## 시스템 흐름

방문자가 드롭다운에서 언어를 고르면, 사이트는 (1) 화면에 뿌릴 문구 사전을 선택 언어로 바꾸고, (2) 문서의 언어 표기와 글 방향(왼→오 또는 오→왼) 속성을 그 언어에 맞게 갱신하며, (3) 선택값을 브라우저 로컬 저장소에 기록한다. 특정 문구가 선택 언어 사전에 없으면 영어 문구로 자연스럽게 대체된다(빈 화면 방지). 첫 진입 시에는 주소의 `lang` 파라미터 → 저장된 선택 → 브라우저 언어 추론 순으로 초기 언어가 결정된다.

## 어디에 구현되어 있나

- `landing/src/components/LocaleSwitcher.tsx` — 헤더의 언어 선택 드롭다운 UI.
- `landing/src/i18n/context.tsx` — 현재 언어 상태 보관, 초기 언어 결정, 선택 저장, 문서 속성 반영, 문구 조회를 담당하는 컨텍스트.
- `landing/src/i18n/locales.ts` — 지원 언어 20종 목록과 표시명·글 방향, 브라우저 언어 추론 규칙.
- `landing/src/i18n/dict.ts` — 언어별 화면 문구 사전.

이 기능은 랜딩 사이트(`component.landing-site`)에 속한 다국어 처리 부품(`component.i18n`)으로 실현된다.

## 미확정 (OPEN)
- [ ] OPEN: 랜딩 다국어 사전과 훅/설치 메시지용 로케일 체인(`integration.i18n-locale`)이 정책상 어디까지 공유·분리되는지는 코드만으로 단정하지 않음(랜딩은 자체 사전 사용 확인).
