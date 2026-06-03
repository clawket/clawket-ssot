---
id: integration.secret-vaults
kind: Integration
title: 비밀 저장소(vault) 연동(env·keyring·1Password·prompt)
definition: 실행 봉투가 비밀 값 대신 담고 있는 "참조"를 여러 백엔드(환경변수·OS 키체인·1Password·대화형 프롬프트)에서 해석해, 도구 실행 직전에만 환경변수로 주입하는 연동이다. 데몬은 대화형 프롬프트 백엔드를 거부한다.
purpose: API 키 같은 민감 값을 봉투나 DB에 평문으로 남기지 않으면서도, 실제 실행 시점에 필요한 비밀을 안전하게 공급하기 위함 — 비밀은 참조로만 저장되고 값은 실행 순간에만 메모리에 존재한다.
integratesWith:
  - component.daemon
implementedIn:
  - daemon/src/secrets/mod.rs
  - daemon/src/secrets/onepassword.rs
impacts:
  - concept.execution-envelope
  - concept.task-envelope
relatesTo:
  - to: invariant.secret-never-in-envelope
    type: relates-to
    note: 봉투에는 비밀 참조만 들어가고 값은 절대 들어가지 않는다
  - to: concept.execution-envelope
    type: relates-to
    note: 봉투의 secrets 참조를 실행 시점에 실제 환경변수로 풀어낸다
governedBy:
  - invariant.secret-never-in-envelope
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

연동 상대는 비밀을 보관하는 여러 외부·로컬 저장소다. 핵심 발상은 "봉투에는 비밀 값이 없다"는 것이다. 봉투는 오직 참조만 담는다 — 어느 백엔드의 어느 경로에서 값을 가져오라는 지시(예: `백엔드:경로` 형태의 문자열 또는 동일한 의미의 객체)다. 실제 값을 가져오는 것은 도구를 실행하기 직전 단 한 번이다. 이때 해석기가 봉투를 훑어 각 참조를 선언된 백엔드에서 꺼내 환경변수로 주입한다.

지원하는 백엔드는 네 가지다.
- 환경변수(env): 프로세스 환경변수를 읽는다. 가볍고 CI에서 쓰기 좋지만 어디에도 영구 저장되지 않는다.
- OS 키체인(keyring): 운영체제 키체인을 통하는 통로(추상 인터페이스 뒤에 실제 키체인 구현이 붙는 구조).
- 1Password(op): 1Password CLI(`op`)를 호출해 `op://Vault/Item/Field` 형태의 경로에서 값을 읽어 온다.
- 대화형 프롬프트(prompt): 사용자에게 직접 입력받는 방식.

중요한 규칙이 하나 있다 — 데몬은 대화형 프롬프트 백엔드를 항상 거부한다. 데몬에는 터미널이 없어 사람에게 물어볼 수단이 없기 때문이며, 프롬프트로 비밀을 받는 것은 CLI 프로세스 안에서만 가능하다. 데몬이 프롬프트 백엔드를 만나면 명확한 오류("프롬프트 백엔드는 데몬에서 거부됨 — env/keyring/1password를 쓰라")로 응답한다.

비밀 값의 수명도 짧게 관리된다. 해석된 값은 로그에 찍어도 별표(`***`)로만 보이는 전용 타입에 담기고, 메모리에서 해제될 때 버퍼를 0으로 덮어 평문이 프로세스 메모리에 남는 시간을 줄인다. 추가로, 봉투를 만들거나 고칠 때 어떤 문자열 값이 키처럼 보이면(엔트로피가 높고 충분히 길면) 실수로 붙여넣은 API 키로 간주해 요청을 거부하는 방어 장치도 함께 있다.

## 구현 위치 (provenance)

해석기·백엔드 추상·비밀 타입은 `daemon/src/secrets/mod.rs`에, 1Password 백엔드 구현은 `daemon/src/secrets/onepassword.rs`에 있다. 데몬(component.daemon)이 도구를 실행하기 직전에 이 해석기를 호출한다. 본문만으로 재현하려면 이렇게 기억하면 된다 — "봉투의 참조를 네 백엔드(env·keyring·op·prompt) 중 선언된 곳에서 풀어 환경변수로 주입하되, 데몬에서는 prompt를 거부하고, 값은 별표 마스킹·메모리 0 채움으로 보호한다."

## 불변식

봉투에는 비밀 값이 절대 들어가지 않고 참조만 들어간다(invariant.secret-never-in-envelope). 이 제약이 연동 전체의 존재 이유다 — 봉투·DB가 노출되어도 비밀 자체는 거기 없고, 값은 실행 시점에만 백엔드에서 잠깐 꺼내 쓰인다. 데몬의 프롬프트 백엔드 거부 역시 이 원칙을 지키는 운영 규칙이다(터미널 없는 환경에서 비밀을 받는 잘못된 경로를 닫는다).

## 영향 범위

이 연동이 닿는 것은 실행 봉투다(concept.execution-envelope, concept.task-envelope). 봉투가 비밀을 다루는 방식·지원 백엔드·거부 규칙이 바뀌면, 실행 시점의 환경변수 주입 동작과 봉투 검증(키처럼 보이는 값 거부)이 함께 달라진다.

## 미확정 (OPEN)
- [ ] OPEN: keyring 백엔드의 운영용 OS 키체인 결선이 실제 production 실행 경로에서 활성화되었는지(현재 일부는 테스트 기본값/미결선 상태로 보임) 확인 필요
