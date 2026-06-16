---
id: concept.hero-version-label
kind: Concept
title: Hero 버전 라벨 (hero.version)
definition: 랜딩 홈 상단에 표시되는 "clawket — vMAJOR.MINOR" 문구. Clawket 플러그인 릴리스 버전의 메이저·마이너를 미러링하는 화면 표시값이다.
relatesTo:
  - to: concept.dict
    type: belongs-to
    note: 버전 라벨은 카피 사전의 한 항목(hero.version)으로 로케일마다 한 줄씩 들어 있다.
  - to: screen.home
    type: relates-to
    note: 홈 화면 최상단 hero 영역에 노출된다.
  - to: platform.claude-code-plugin
    type: relates-to
    note: 표시되는 버전은 Clawket 플러그인 릴리스 태그의 major.minor 를 따라간다.
governedBy: []
governs: []
dependsOn:
  - concept.dict
implementedIn:
  - landing/src/i18n/dict.ts
  - landing/scripts/update-version-label.sh
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 식별자는 provenance/근거에만. -->

## 정의

Hero 버전 라벨은 랜딩 홈 화면 맨 위 hero 영역에 작게 표시되는 "clawket — vMAJOR.MINOR"(예: "clawket — v3.1") 문구다. 방문자가 지금 보는 사이트가 어느 버전대의 Clawket 을 소개하는지 한눈에 알리는 표지다.

이 라벨은 별도의 위젯이 아니라 [카피 사전](dict.md)의 한 항목(`hero.version`)일 뿐이다. 따라서 20개 [로케일](locale.md)마다 같은 라벨 줄이 한 개씩 존재한다. 다만 버전 표기는 언어 중립적이어서, 어느 언어로 보든 같은 "clawket — v3.1" 형태로 보인다(언어별로 다른 것은 라벨이 아니라 그 주변 카피들이다).

라벨이 담는 것은 메이저·마이너 두 자리(예: v3.1)까지이고 패치 번호는 의도적으로 싣지 않는다. 랜딩은 "이번 버전 라인"의 기준선만 알리는 자리이고, 패치 단위 갱신마다 마케팅 사이트를 다시 배포할 필요가 없도록 한 결정이다.

진실의 출처는 랜딩이 아니다. 버전의 단일 출처는 Clawket 플러그인 저장소의 릴리스 태그이며(같은 메이저·마이너로 CLI·데몬·웹 산출물 핀이 함께 움직인다 — [platform.claude-code-plugin](../platform.md) 참조), 랜딩의 라벨은 그것을 사람이 손으로 미러링한 사본이다. 플러그인 릴리스 절차의 한 단계로 갱신 스크립트를 직접 돌려 20개 로케일의 라벨 줄을 한꺼번에 같은 값으로 맞춘다. 이 스크립트는 빌드 과정에서 자동 호출되지 않는다 — 랜딩 빌드를 오프라인·결정적으로 유지하기 위해 수동 단계로 둔다.

## 엔티티 (DB)

데이터베이스 엔티티가 아니다. 라벨은 카피 사전 소스 안의 한 줄(로케일마다 한 개)이며 빌드 시 번들에 포함된다. 런타임에 외부에서 버전을 받아오지 않는다.

## API 표면

서버 API 가 없다. 라벨은 정적 문자열로 화면에 렌더되며, 어떤 버전 조회 호출도 발생시키지 않는다.

## 불변식

- 표시 형식은 항상 "clawket — vMAJOR.MINOR"이며 패치 번호는 포함하지 않는다(메이저.마이너 기준선만 노출).
- 갱신 시 모든 로케일의 라벨 줄이 같은 값으로 일괄 변경된다 — 일부 언어만 다른 버전을 표기하는 상태는 허용되지 않는다(갱신 도구가 변경 전후 매칭 행 수가 일치하는지 검증하고, 불일치 시 실패한다).
- 표시값은 Clawket 플러그인 릴리스 태그의 메이저·마이너를 따라간다(랜딩이 독자적으로 버전을 정하지 않는다).

## 구현 위치 (provenance)

라벨 값 자체는 `landing/src/i18n/dict.ts` 의 각 로케일 항목에 `hero.version` 키로 들어 있다. 플러그인 태그에 맞춰 모든 로케일의 라벨을 일괄 치환·검증하는 갱신 도구는 `landing/scripts/update-version-label.sh` 다. 코드가 사라져도, "메이저.마이너만 / 모든 로케일 동일 / 플러그인 태그를 사람이 미러링 / 빌드 자동화 아님" 규칙만으로 동작을 재현할 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: 플러그인 릴리스와 랜딩 라벨 갱신 사이의 시점 차로 인한 일시적 버전 불일치 허용 범위가 명시되지 않음.
