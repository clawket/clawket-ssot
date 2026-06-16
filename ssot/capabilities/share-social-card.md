---
id: capability.share-social-card
kind: Capability
title: 링크 공유 카드(소셜 미리보기)
definition: 사이트의 각 라우트가 자신에 맞는 제목·설명·정규 주소와 오픈그래프·트위터 메타데이터를 화면 진입 시 문서 head에 반영해, 링크를 공유했을 때 검색·SNS에서 올바른 미리보기 카드와 대표 이미지가 표시되도록 한다.
purpose: 링크가 공유되거나 검색에 노출될 때 페이지별로 정확한 제목·설명·대표 이미지가 보여 클릭과 신뢰를 높인다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.landing-site
  - component.og-pipeline
implementedIn:
  - landing/src/hooks/useHead.ts
  - landing/public/og-image.png
relatesTo:
  - to: endpoint.asset-og-image
    type: relates-to
    note: 공유 카드의 대표 이미지가 이 정적 자산으로 제공된다
  - to: endpoint.route-docs
    type: relates-to
    note: 문서 허브를 포함한 각 라우트가 자신의 head 정보를 설정한다
impacts:
  - endpoint.asset-og-image
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

이 기능은 방문자가 직접 조작하는 것이 아니라, 누군가 사이트 링크를 메신저·SNS·검색에 올렸을 때 자동으로 작동한다. 각 페이지(홈, 문서 허브, 시작하기·워크플로우·CLI·아키텍처)는 자기 내용에 맞는 제목·설명·정규 주소와, 오픈그래프·트위터용 미리보기 정보를 갖고 있다. 따라서 어느 페이지의 링크를 공유하더라도 그 페이지에 맞는 제목·설명과 1200×630 규격의 대표 이미지가 미리보기 카드로 노출되어, 받는 사람이 무슨 페이지인지 한눈에 안다.

## 행위

- 설정: 라우트 진입 시 문서 제목을 해당 페이지 제목으로 바꾼다.
- 설정: 설명 메타데이터, 정규 주소 링크, 오픈그래프(제목·설명·주소), 트위터(제목·설명) 메타데이터를 해당 페이지 값으로 갱신한다.
- 폴백: 초기 HTML에는 기본 제목·설명·정규 주소·오픈그래프·트위터 블록이 들어 있어, 자바스크립트를 실행하지 않는 크롤러도 최소한의 미리보기를 얻는다.

이 기능은 데몬 API나 CLI를 호출하지 않고 문서 head만 갱신한다.

## 시스템 흐름

방문자가 어느 라우트로 진입하면, 그 페이지가 자신의 제목·설명·정규 주소·공유용 주소를 head 갱신 훅에 넘긴다. 훅은 문서 head에서 해당 메타 요소를 찾아 없으면 만들고, 있으면 값을 덮어써 제목·설명·정규 링크·오픈그래프·트위터 정보를 페이지에 맞게 동기화한다. 대표 이미지는 사이트의 정적 자산(`endpoint.asset-og-image`)으로 제공된다. 라우트 전환 사이의 빈 head 깜빡임을 막기 위해 문서 영역은 무해한 임시 폴백 값을 둔다.

## 어디에 구현되어 있나

- `landing/src/hooks/useHead.ts` — 라우트별 제목·설명·정규 주소·오픈그래프·트위터 메타데이터를 문서 head에 반영하는 훅, 그리고 사이트 기준 주소(origin).
- `landing/public/og-image.png` — 공유 카드의 대표 이미지(1200×630 규격).
- 각 페이지 컴포넌트(홈·문서 허브·네 섹션)가 이 훅을 호출해 자신의 head 값을 설정한다.

이 기능은 랜딩 사이트(`component.landing-site`)에서 실현되며, 대표 이미지 생성·제공은 OG 파이프라인 부품(`component.og-pipeline`)과 연계된다.

## 미확정 (OPEN)
- [ ] OPEN: 1200×630 규격은 인벤토리 설명에 기재된 값으로, 이미지 자산의 실제 해상도는 코드만으로 검증되지 않음(자산 메타데이터 확인 필요).
