---
id: concept.dict
kind: Concept
title: Dict (카피 사전)
definition: 로케일 하나당 한 벌씩 존재하는, 화면 카피 문구들의 키-값 모음. 랜딩 홈 화면이 보여주는 모든 텍스트의 단일 출처다.
relatesTo:
  - to: concept.locale
    type: relates-to
    note: 로케일마다 사전 한 벌이 대응한다 — 로케일이 사전을 고르는 키.
  - to: concept.hero-version-label
    type: contains
    note: 홈 상단 버전 라벨도 이 사전의 한 항목(hero.version)으로 들어 있다.
  - to: component.i18n
    type: belongs-to
    note: 사전과 그 조회 함수는 자체 i18n 엔진의 일부다.
  - to: screen.home
    type: relates-to
    note: 홈 화면의 카피가 전부 이 사전에서 온다.
governedBy: []
governs: []
dependsOn:
  - concept.locale
implementedIn:
  - landing/src/i18n/dict.ts
  - landing/src/i18n/context.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 식별자는 provenance/근거에만. -->

## 정의

Dict(카피 사전)은 랜딩 사이트가 화면에 띄우는 문구들을 "의미 키 → 번역된 문장" 형태로 묶은 모음이다. 사이트가 지원하는 20개 [로케일](locale.md) 각각에 대해 같은 키 집합을 채운 사전이 한 벌씩 있다. 즉, 영어 사전·한국어 사전·일본어 사전 … 이 나란히 존재하고, 키는 모든 언어에서 동일하며 값(문장)만 언어별로 다르다.

사전이 담는 키는 28개다. 이들은 홈 화면의 구성 요소와 1:1로 대응한다: 상단 버전 라벨(`hero.version`), 헤드라인 두 줄과 리드 문단, 두 개의 CTA 버튼("마켓플레이스에서 설치", "GitHub 조직"), 데모 섹션 제목·접근성 설명, 워크플로 강제 방식 섹션의 제목·태그라인과 3단계(활성 태스크 게이트 / done 시 evidence 강제 / 완료 시 cascade)의 제목·본문, 기능 섹션 제목과 3개 기능 카드, 아키텍처 섹션 제목·본문, 푸터 문구, 언어 선택기 라벨이다. 일부 본문 값에는 `<code>…</code>` 같은 인라인 코드 표시 마크업이 들어 있어, 화면에서는 코드 스타일로 강조되어 렌더된다.

문구를 꺼내 쓰는 규칙은 단순한 폴백 체인이다: 현재 로케일의 사전에서 키를 찾고, 그 로케일에 사전이 없거나 해당 키가 비어 있으면 영어 사전의 값으로 떨어지며, 그것마저 없으면 키 이름 자체를 반환한다. 덕분에 번역이 일부 누락돼도 화면이 비거나 깨지지 않고 최소한 영어로 보인다.

이웃 개념: 사전은 [concept.locale](locale.md) 로 선택되고(로케일=키), 그 안에 [concept.hero-version-label](hero-version-label.md)(버전 라벨)을 한 항목으로 품으며, [component.i18n](../components/i18n.md) 자체 i18n 엔진의 일부로서 [screen.home](../screens/home.md) 홈 화면 카피의 단일 출처가 된다.

## 엔티티 (DB)

데이터베이스 엔티티가 아니다. 사전은 프런트엔드 소스 코드 안에 박힌 정적 상수 구조(로케일별 키-값 표)이며, 빌드 시점에 번들에 포함되어 배포된다. 런타임에 외부에서 받아오거나 사용자가 편집하는 데이터가 아니다.

## API 표면

서버 API 가 없다. 사전 조회는 클라이언트 안에서 메모리 상수를 읽는 동작이며 네트워크 호출을 동반하지 않는다. 화면 컴포넌트는 i18n 컨텍스트가 제공하는 번역 함수와 인라인 렌더 헬퍼를 통해 키로 문구를 조회한다.

## 불변식

- 모든 로케일 사전은 동일한 키 집합(현재 28개)을 가져야 한다 — 키가 누락된 로케일이 있어도 조회는 영어 폴백으로 메워진다.
- 조회는 "현재 로케일 → 영어 → 키 이름" 폴백 체인을 따르며, 어떤 경우에도 빈 화면을 내지 않는다.
- `hero.version` 값의 형식("clawket — vMAJOR.MINOR")은 모든 로케일에서 동일하게 유지된다(버전 라벨 갱신 도구가 이 형식을 전제로 일괄 치환한다).

## 구현 위치 (provenance)

로케일별 카피 사전 전체와 키 정의는 `landing/src/i18n/dict.ts` 에 있다. 폴백 체인을 적용해 문구를 꺼내는 조회 함수와 인라인 코드 마크업을 화면 요소로 변환하는 헬퍼는 `landing/src/i18n/context.tsx` 에 있다. 코드가 사라져도, 위에 적은 키 목록(28개)·로케일별 한 벌 구조·"현재→영어→키이름" 폴백 규칙만으로 동작을 재현할 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: 키가 추가/삭제될 때 20개 로케일을 동시에 채우도록 강제하는 절차의 소유 주체가 명시되지 않음(owner TBD).
