---
id: integration.i18n-locale
kind: Integration
title: 다국어 로케일 체인
purpose: "사용자가 한국어·일본어 환경에서도 훅 가드 메시지와 설치 안내를 모국어로 받게 해, 영어 외 사용자가 차단 사유·해결 방법을 즉시 이해하도록 한다."
definition: "환경변수 기반으로 활성 로케일을 정하고, 로케일별 fallback 순서(예: ja→ko→en)를 따라 메시지 카탈로그에서 텍스트를 찾아 현지화하는 CLI/훅 런타임 다국어 체계."
integratesWith:
  - component.claude-adapter
implementedIn:
  - clawket/adapters/shared/locale.cjs
  - clawket/adapters/shared/claude-hooks.cjs
  - clawket/locales
  - clawket/docs/i18n-policy.md
impacts:
  - endpoint.hook-session-start
  - endpoint.hook-user-prompt-submit
  - endpoint.hook-pre-tool-use
  - endpoint.hook-post-tool-use
  - endpoint.hook-plan-sync
  - component.install-gate
  - integration.git-destructive-guard
relatesTo:
  - to: integration.git-destructive-guard
    type: relates-to
    note: "파괴적 명령 가드의 차단 메시지(이유·해결책)가 이 로케일 카탈로그를 통해 현지화되며, 패턴 정의 자체도 로케일별 문구 테이블을 갖는다."
  - to: component.install-gate
    type: relates-to
    note: "설치 게이트가 출력하는 진행·경고·실패 안내 메시지가 로케일 체인을 거쳐 현지화된다."
  - to: integration.github-releases
    type: relates-to
    note: "릴리스 다운로드/레이트리밋 등 설치 경로 메시지가 로케일 키로 관리된다."
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance/근거에만). 분기/의도 불명은 OPEN. -->

## 무엇과 연동하나

이 연동은 외부 서비스가 아니라 **사용자의 셸 환경(로케일 환경변수)** 과의 연동이다. Clawket의 명령줄 도구와 Claude Code 훅은 사용자에게 짧은 안내·차단·설치 메시지를 출력하는데, 이 메시지들을 사용자의 언어 환경에 맞춰 한국어·일본어·영어 중 하나로 골라 보여 준다.

활성 언어는 사용자가 따로 정해 주지 않아도 환경에서 자동으로 추론된다. 추론 순서는 다음과 같고, 먼저 맞는 것이 이긴다.

1. Clawket 전용 언어 지정 환경변수
2. 시스템 전역 로케일 설정(있으면 지역·인코딩 꼬리표를 떼어 `ko_KR.UTF-8`→`ko`처럼 언어 코드만 취함)
3. 일반 언어 환경변수(같은 방식으로 언어 코드만 취함)
4. 어느 것도 없으면 영어

지원하는 언어는 영어·한국어·일본어 세 가지다. 지원하지 않는 값이 명시적으로 지정되면, 사용자가 오타를 알아챌 수 있도록 한 번만 경고를 띄운 뒤 영어로 떨어진다(조용히 영어로 바뀌어 혼란을 주지 않는다).

핵심 동작은 **로케일 fallback 체인**이다. 어떤 언어를 골랐더라도, 그 언어 카탈로그에 해당 문구가 빠져 있으면 다음 순위 언어로 넘어가 찾는다. 체인의 끝은 항상 영어라서, 영어 카탈로그에 존재하는 문구는 어떤 언어를 골라도 반드시 무언가는 표시된다. 특히 일본어는 `ja→ko→en` 순서를 쓴다 — 일본어 문구가 비어 있을 때 영어보다 한국어가 표기·의미상 더 가깝다는 판단에 따른 것이다. 한국어는 `ko→en`, 영어는 영어 자신뿐이다. 모든 단계에서 문구를 못 찾으면, 호출하는 쪽이 항상 비지 않은 문자열을 받도록 메시지 키 자체를 그대로 돌려준다.

## 구현 위치 (provenance)

런타임 본체는 `clawket/adapters/shared/locale.cjs`다. 여기서 환경변수로부터 활성 로케일을 추론하고(`resolveLocale`), 로케일별 fallback 순서를 정의하며(`localeChain`), 키를 받아 체인을 따라 카탈로그를 조회해 문구를 반환한다(`t`). 카탈로그는 로케일별 JSON 파일(`clawket/locales/en.json`·`ko.json`·`ja.json`)에 키-문구 쌍으로 들어 있으며, 한 번 읽으면 메모리에 캐시된다.

이 런타임을 실제로 쓰는 내부 컴포넌트는 Claude 어댑터(`clawket/adapters/shared/claude-hooks.cjs`)다. 어댑터는 첫 호출 때 한 번만 로케일을 추론해 캐시하고(훅 한 번당 환경변수 읽기 1회), 조용히 모드가 아니면 어떤 언어와 어떤 fallback 체인이 적용되는지 한 줄로 알려 준다. 가드 차단 메시지·플랜 동기화 안내·설치 진행/경고/실패 메시지 등 사용자에게 보이는 모든 텍스트가 이 경로를 거친다.

진단을 위한 보조 동작도 있다. 디버그 모드에서는 (1) 어떤 키가 체인을 따라 다른 언어로 떨어졌는지 한 줄씩 알려 주고, (2) 부팅 시 일본어·한국어 카탈로그가 영어 대비 빠진 키가 있는지 점검해 경고한다. 이 점검은 디버그 모드가 아니면 아무 일도 하지 않으며, 빠진 키가 있어도 사용자 세션을 중단시키지 않는다.

정책 문맥은 `clawket/docs/i18n-policy.md`에 정리돼 있다. 이 정책은 세 축을 구분한다 — 문서 번역(영어+한국어, 드리프트 추적 대상), **여기서 다루는 CLI 런타임 문구**(영어+한국어+일본어, 호출마다 환경변수로 해석, 빠진 키는 체인으로 fallback), 그리고 별도의 랜딩 페이지 런타임 문구(20개 언어, 본 연동과 무관한 별도 사전). 즉 본 연동은 그중 **CLI/훅 런타임 축** 하나에 해당한다. 네 번째 CLI 런타임 언어를 추가하는 일은 문서 작업이 아니라 코드 변경(JSON 번들 추가 + 지원 목록 확장)이다.

## 불변식

- 체인의 마지막은 항상 영어다. 따라서 영어 카탈로그에 존재하는 키는 어떤 활성 언어에서도 반드시 무언가로 표시된다.
- 모든 fallback을 거쳐도 못 찾으면 키 문자열 자체를 반환한다 — 호출 측은 결코 빈 문자열/누락 텍스트를 받지 않는다.
- 지원하지 않는 언어가 명시되면 한 번만 경고하고 영어로 떨어진다(조용한 영어 전환 금지). 같은 오설정에 대해 한 훅 프로세스 안에서 경고는 중복되지 않는다.
- 활성 언어는 프로세스 범위로 한 번 결정·캐시된다(훅 한 번 = 언어 1회 결정).

이 연동에 별도로 등록된 invariant.* 노드는 확인되지 않았다(governedBy 비어 있음).

## 영향 범위

로케일 체계가 닿는 표면은 사용자에게 메시지를 출력하는 모든 훅과 설치/가드 경로다. 세션 시작 컨텍스트 주입, 사용자 프롬프트 제출 경고, 변경 차단(PreToolUse) 가드 메시지, 파일 변경 기록(PostToolUse), 플랜 동기화 안내가 모두 이 카탈로그의 문구를 쓴다. 설치 게이트의 진행·레이트리밋·실패 안내, 그리고 파괴적 명령 가드의 차단 사유·해결책 문구도 동일 경로로 현지화된다. 따라서 메시지 키를 바꾸거나 카탈로그에 키를 추가/삭제하면 위 표면 전체의 표시 텍스트가 영향을 받는다.

## 미확정 (OPEN)
- [ ] OPEN: CLI 본체(Rust)의 출력 메시지가 이 `.cjs` 카탈로그와 별개의 로케일 경로(`daemon/src/locale.rs` 등)를 쓰는지, 두 축의 키가 어떻게 정합되는지 — 본 노드 범위(훅/어댑터 런타임) 밖이라 미확정.
- [ ] OPEN: 이 연동에 직접 걸린 invariant.* 노드 존재 여부(governedBy) 확정 필요.
