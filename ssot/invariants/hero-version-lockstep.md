---
id: invariant.hero-version-lockstep
kind: Invariant
title: Hero 버전 라벨 일괄 갱신
definition: "홈 상단의 'clawket — vMAJOR.MINOR' 버전 라벨은 릴리스 때 전용 스크립트가 20개 로케일의 라벨 행을 한 번에 동일 값으로 치환하는 방식으로만 갱신하며, 개별 로케일을 손으로 따로 고치지 않는다."
governs:
  - concept.hero-version-label
  - concept.dict
  - component.i18n
  - screen.home
governs_note: ""
implementedIn:
  - landing/scripts/update-version-label.sh
  - landing/src/i18n/dict.ts
  - landing/CLAUDE.md
decidedBy:
  - decision.real-product-hero
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:invariant"]
---

## 제약

홈 화면 상단에 보이는 버전 라벨("clawket — vMAJOR.MINOR")은 20개 지원 언어 카피 사전에 언어마다 한 행씩 들어 있다. 이 라벨을 바꿀 때는 전용 갱신 스크립트를 한 번 실행해 20개 로케일 행을 모두 동일한 값으로 동시에(lock-step) 치환해야 한다. 특정 언어 행만 손으로 따로 고치는 것은 금지된다 — 표시되는 버전은 언어와 무관하게 항상 같아야 하기 때문이다.

라벨 값은 clawket 플러그인 태그의 메이저·마이너 버전을 미러한다(패치 단위는 표시하지 않음). 즉 라벨의 단일 진실 출처는 플러그인 릴리스 태그이고, 라벨 갱신은 그 릴리스 절차의 한 단계로 수행된다. 또한 이 갱신은 빌드 과정에 끼워 넣지 않는다 — 빌드는 결정론적이고 오프라인이어야 하므로, 라벨 치환은 릴리스 때 수동으로 한 번 돌리는 별도 단계다.

## 깨지면 무슨 일이 일어나나

일부 로케일만 갱신되면 같은 사이트가 언어에 따라 서로 다른 버전을 표시하게 되어, 방문자가 보는 제품 버전이 어긋난다. 라벨을 손으로 개별 수정하다 보면 누락·오타가 끼고, 표시 버전이 실제 플러그인 릴리스와 일치하지 않게 된다. 라벨의 형식이나 위치를 바꾸는 변경을 하면서 갱신 스크립트의 매칭 패턴을 함께 손보지 않으면, 다음 릴리스 때 스크립트가 한 행도 못 찾거나 일부만 바꿔 갱신이 불완전해진다(스크립트 자체가 갱신 전후 행 수를 비교해 불일치 시 실패하도록 되어 있다).

## 코드에서 어떻게 강제되나

갱신 스크립트(`landing/scripts/update-version-label.sh`)가 카피 사전(`landing/src/i18n/dict.ts`)의 모든 로케일 라벨 행을 한 번에 in-place 치환한다. 스크립트는 입력 버전을 메이저.마이너로 정규화하고, 치환 전 매칭된 행 수를 세어 두었다가 치환 후 같은 수의 행이 새 값으로 바뀌었는지 검증해, 불일치하면 실패하고 종료한다. 갱신을 빌드가 아니라 릴리스 수동 단계로 돌린다는 점과 개별 로케일 손수정 금지는 작성 규칙(`landing/CLAUDE.md`)에 명시되어 있다.

라벨 값이 실제 플러그인 릴리스 태그와 일치하는지를 자동으로 검사하는 장치는 없다 — 릴리스 절차에서 사람이 스크립트를 빠뜨리지 않고 실행하는 데 의존한다. 라벨 형식·위치를 바꿀 때 스크립트의 매칭 패턴·대상 파일 변수도 함께 갱신하라는 점도 작성 규칙으로만 보존된다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy 는 "hero 가 실제 제품 표면"이라는 결정에 연결했다(버전 라벨 정합이 그 신뢰성의 일부). 라벨 일괄 갱신 방식 자체에 대한 전용 결정이 필요하면 추가.
