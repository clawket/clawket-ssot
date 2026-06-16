---
id: integration.compliance-evals-feed
kind: Integration
title: 모델 등급별 준수율 피드(랜딩 배지, 미배선)
purpose: "모델 등급(haiku/sonnet/opus)별로 실행 계약을 얼마나 잘 지키는지를 외부에 게시된 최신 스냅샷에서 가져와 랜딩 페이지에 신뢰 지표로 보여 주려는 의도."
definition: "외부에 게시된 준수율 스냅샷(JSON)을 랜딩 페이지의 배지 컴포넌트가 가져와(fetch) 등급별 퍼센트와 게이트 통과 여부를 표시하는 외부 데이터 연동. 현재 어느 페이지에도 연결되어 있지 않다(미배선)."
integratesWith: []
implementedIn:
  - landing/src/components/ComplianceBadge.tsx
impacts: []
relatesTo:
  - { to: concept.tier, type: relates-to, note: "스냅샷의 세 수치는 모델 등급(haiku/sonnet/opus) 단위의 준수율로, clawket 의 티어 개념을 그대로 따른다." }
  - { to: concept.compute-tier, type: relates-to, note: "동일한 모델 등급 개념의 다른 표현 — 등급별 준수율이라는 지표가 가리키는 대상." }
governedBy: []
owner: TBD
lifecycle: proposed
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:integration"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드/식별자는 provenance·근거에만. -->

## 무엇과 연동하나

랜딩 페이지에는 모델 등급별로 "실행 계약을 얼마나 잘 지키는가" 를 보여 주려는 작은 배지 위젯이 준비되어 있다. 이 위젯은 외부에 게시된 평가 결과 스냅샷(공개 정적 호스팅 위의 JSON 파일)을 인터넷에서 받아와, 세 등급(haiku·sonnet·opus)의 준수율 퍼센트와 게이트(기준 통과 여부) 표시를 그린다. 받아오는 동안에는 "로딩", 실패하면 "이용 불가", 성공하면 등급별 수치와 갱신 시각을 보여 준다. 받은 데이터가 기대한 형태가 아니면 오류로 처리해 잘못된 수치를 그리지 않는다.

중요한 사실: **이 위젯은 현재 랜딩의 어느 페이지에도 실제로 붙어 있지 않다.** 컴포넌트와 그 테스트만 존재하고, 실제 화면에 노출되는 배선이 아직 없다 — 향후 배선하거나 제거할 후보다. 그래서 이 연동은 "의도는 있으나 아직 살아 있는 경로가 아닌" 검토 단계로 본다.

## 구현 위치 (provenance)

위젯은 랜딩 저장소의 배지 컴포넌트(`landing/src/components/ComplianceBadge.tsx`)에 구현되어 있다. 데이터 출처 주소는 컴포넌트 내부에 기본값으로 박힌, clawket 조직의 공개 정적 호스팅(GitHub Pages 성격) 위에 게시되는 최신 스냅샷 파일이다(주소는 호출 시 다른 값으로 바꿔 넣을 수도 있다). 이 데이터를 게시하는 쪽(평가 파이프라인)이 어디서 어떻게 스냅샷을 만들어 올리는지는 랜딩 저장소만으로는 확인되지 않으며, 본 SSOT 인벤토리에도 그 게시 측 컴포넌트 노드가 없어 integratesWith 는 비워 둔다(OPEN).

스냅샷이 담는 세 수치는 모델 등급(haiku/sonnet/opus)별 준수율로, clawket 의 등급(티어) 개념을 그대로 따른다(relatesTo 참조).

## 불변식

- 받은 스냅샷이 기대한 형태(갱신 시각·스키마 버전·세 등급 수치)를 갖추지 않으면 오류로 처리해, 검증되지 않은 수치를 화면에 그리지 않는다.
- 데이터는 캐시 없이 매번 새로 받아 최신 스냅샷을 반영한다.

## 영향 범위

이 연동은 아직 어느 화면에도 배선되지 않은 검토 단계이므로, 현재 운영 표면에 미치는 영향은 없다. 배선될 경우 랜딩 페이지의 신뢰 지표 표시에 영향을 주며, 표시 대상은 clawket 의 모델 등급 개념을 가리킨다. 본 SSOT 인벤토리 등재 노드 중 직접 영향을 받는 대상은 확인되지 않았다(impacts 는 OPEN).

## 미확정 (OPEN)
- [ ] OPEN: 준수율 스냅샷을 게시하는 평가 파이프라인(데이터 게시 측)이 SSOT 인벤토리에 노드로 없어 integratesWith 미배선
- [ ] OPEN: 위젯의 실제 페이지 배선 여부·시점 미정 — 현재 미배선
- [ ] OPEN: 영향 대상(impacts) 확정 필요
