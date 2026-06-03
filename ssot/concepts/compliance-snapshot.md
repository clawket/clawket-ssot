---
id: concept.compliance-snapshot
kind: Concept
title: 계약 준수 스냅샷(Compliance Snapshot)
definition: "모델 등급별(haiku/sonnet/opus) 계약 준수율과 출시 게이트 통과 여부를 담아 외부에서 주기적으로 갱신·게시하는 평가 결과 데이터 형. 랜딩 사이트의 배지 컴포넌트가 이 데이터를 가져와 표시하도록 설계됐으나, 현재는 어떤 화면에도 배선되지 않고 테스트로만 존재한다."
relatesTo:
  - { to: integration.compliance-evals-feed, type: backed-by, note: "외부 평가 피드(latest.json)가 이 스냅샷을 산출·게시한다" }
  - { to: concept.tier, type: reads, note: "haiku/sonnet/opus 세 등급은 모델 티어 개념과 같은 등급 축을 가리킨다" }
  - { to: component.landing-site, type: belongs-to, note: "랜딩 사이트의 배지 컴포넌트가 이 데이터를 소비하도록 만들어졌다" }
  - { to: screen.home, type: relates-to, note: "히어로 화면에 노출하려는 용도였으나 현재는 미배선" }
governedBy: []
dependsOn:
  - integration.compliance-evals-feed
implementedIn:
  - landing/src/components/ComplianceBadge.tsx
  - landing/src/components/ComplianceBadge.test.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

계약 준수 스냅샷은 "이 제품(에이전트)이 약속한 동작 계약을 모델 등급마다 얼마나 잘 지키는가"를 한 장으로 요약한 외부 평가 결과다. 한 스냅샷에는 마지막 갱신 시각, 스키마 버전, 그리고 세 모델 등급(haiku · sonnet · opus) 각각의 준수율(퍼센트), 그리고 선택적으로 "출시 게이트"의 이름과 통과 여부가 들어간다.

여기서 세 등급은 이 제품이 작업에 요구하는 모델 등급(low/med/high에 대응하는 Haiku급·Sonnet급·Opus급)과 같은 축이다. 즉 더 약한 모델에서도 계약이 얼마나 지켜지는지를 보여주는 신뢰 지표 성격을 가진다. 게이트는 특정 출시 마일스톤(예: 안정화 단계)에 도달했는지를 나타내며, 통과했을 때만 시각적으로 강조된다.

이 데이터는 제품 내부 상태가 아니라 **외부에서 주기적으로 측정·게시되는 평가 결과**라는 점이 핵심이다. 랜딩 사이트는 이를 직접 계산하지 않고, 게시된 결과를 가져와(읽기 전용) 표시만 하도록 설계됐다.

## 엔티티 (DB)

이 개념은 데몬의 SQLite에 저장되는 작업 관리 엔티티가 아니다. 데이터베이스 테이블을 갖지 않으며, 외부에 게시된 정적 JSON 문서로만 존재한다. 한 스냅샷 문서는 다음 의미 항목으로 구성된다:

- 갱신 시각: 이 평가가 마지막으로 측정된 시점.
- 스키마 버전: 데이터 형식의 버전. 소비 측은 이 버전이 기대치와 다르면 해석을 거부한다.
- 등급별 준수율: haiku/sonnet/opus 세 등급의 준수 퍼센트(정수).
- 게이트(선택): 출시 게이트의 이름과 통과 여부. 통과했을 때만 강조 표시된다.

소비 측은 위 필수 항목(갱신 시각·스키마 버전·세 등급 준수율)이 모두 올바른 형이어야만 유효한 스냅샷으로 인정하고, 그렇지 않으면 "사용 불가" 상태로 떨어진다.

## API 표면

이 개념을 읽는 표면은 단 하나다 — 랜딩 사이트의 배지 컴포넌트가 외부 평가 피드 URL에서 캐시 없이(no-store) 가져온다. 데몬 HTTP API나 CLI는 이 개념을 다루지 않는다.

소비는 세 가지 표시 상태를 가진다:
- 로딩: 가져오기가 끝나기 전.
- 준비됨: 유효한 스냅샷을 받아 등급별 퍼센트와 게이트 라벨을 표시.
- 사용 불가(에러): HTTP 실패 또는 스키마 불일치 시 조용히 "unavailable"로 대체.

**현재 배선 상태**: 이 배지 컴포넌트는 랜딩의 어떤 화면(히어로 포함)에도 실제로 렌더되지 않으며, 동작은 단위 테스트로만 검증되어 있다. 즉 제품 약속으로 노출되기 전의 준비된 부품 상태다.

## 불변식

- 소비 측은 스키마 버전과 필수 필드 형을 검사해 통과한 데이터만 "준비됨"으로 표시한다 — 형이 맞지 않으면 표시를 거부하고 사용 불가로 떨어진다. (스냅샷 무결성 자기보호)
- 외부에서 데이터를 가져올 때 캐시를 쓰지 않아 항상 최신 게시본을 반영한다.

명시적으로 SSOT에 등록된 invariant 노드와의 직접 연결은 확인되지 않았다(OPEN).

## 구현 위치 (provenance)

- `landing/src/components/ComplianceBadge.tsx`: 외부 피드에서 스냅샷을 가져와 로딩/준비됨/사용 불가 세 상태로 렌더하는 배지 컴포넌트. 기본 피드 URL과 스냅샷 형 검사, 게이트 통과 강조 로직이 여기 있다.
- `landing/src/components/ComplianceBadge.test.tsx`: 로딩·성공·게이트 미통과·HTTP 실패·스키마 불일치·시각 표기 등 동작을 고정한 단위 테스트. 현재 이 개념의 동작이 검증되는 유일한 곳이다.

## 미확정 (OPEN)
- [ ] OPEN: 배지가 실제로 어느 화면에 언제 배선될지(히어로 노출 계획) 미정.
- [ ] OPEN: 외부 평가 피드의 게시 주기·산출 파이프라인 세부는 본 레포 밖이라 미확인.
- [ ] OPEN: 이 스냅샷을 강제하는 명시적 invariant 노드 존재 여부 미확인.
