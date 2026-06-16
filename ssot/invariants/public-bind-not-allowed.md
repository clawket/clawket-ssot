---
id: invariant.public-bind-not-allowed
kind: Invariant
title: 데몬은 로컬 전용 — 외부 노출 바인드 거부
definition: clawketd 데몬은 루프백 주소에만 바인딩하며, 운영자가 명시적으로 동의하지 않는 한 LAN·공개 주소로의 바인드를 시작 단계에서 거부한다.
governs:
  - component.daemon
  - platform.local-first
  - component.sqlite-storage
implementedIn:
  - daemon/src/main.rs
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

## 제약

Clawket 데몬은 본질적으로 한 대의 머신에서 한 사람(과 그 사람의 LLM 에이전트)이 쓰는 로컬 도구다. 저장소인 SQLite 는 여러 사용자가 네트워크 너머에서 동시에 접근하는 상황을 전제로 설계되지 않았다. 따라서 데몬이 받아들이는 네트워크 주소는 **자기 자신(루프백)** 으로 한정된다.

구체적으로, 데몬이 듣겠다고 지정한 호스트가 `127.0.0.1`, `::1`, `localhost`, 혹은 `127.0.0.0/8` 대역의 어떤 주소도 아니라면 — 즉 `0.0.0.0`, 사내망 IP, 공개 IP 처럼 외부에서 닿을 수 있는 주소라면 — 데몬은 기동 자체를 멈춘다. 유일한 예외는 운영자가 환경 변수로 "외부 노출을 알면서도 허용한다"고 명시적으로 opt-in 한 경우뿐이다. 이 동의가 없으면 외부 바인드는 허용되지 않는다.

## 깨지면 무슨 일이 일어나나

이 제약이 무너지면 로컬 전용이라는 전제 아래 설계된 두 가지 안전 장치가 동시에 의미를 잃는다.

첫째, 같은 머신을 신뢰 경계로 보고 유닉스 소켓 연결에 인증을 면제하는 정책(→ invariant.tcp-auth)이 위험해진다. 데몬이 외부 주소에 떠 있으면, 로컬=신뢰라는 가정이 더 이상 성립하지 않는 표면이 생긴다.

둘째, 어떤 데이터도 기기 밖으로 나가지 않는다는 로컬 우선 약속이 깨진다. 인증·임베딩까지 온디바이스로 처리하며 외부 네트워크 경로를 GitHub Releases 다운로드 하나로 제한한 설계 의도(→ platform.local-first)가 무력화된다. 작업 상태·지식·감사 기록이 담긴 SQLite 저장소가 네트워크 너머로 노출될 수 있다.

## 코드에서 어떻게 강제되나

데몬 기동 경로에서 리스너를 열기 직전에, 지정된 호스트가 루프백인지 판정한다. 루프백이 아니면서 공개 바인드 동의 환경 변수가 켜져 있지 않으면, `PUBLIC_BIND_NOT_ALLOWED` 코드와 함께 기동을 중단(bail)한다. 즉 잘못된 주소로 떠 버린 뒤 막는 게 아니라, 애초에 뜨지 못하게 막는 fail-fast 방식이다. 루프백 판정은 잘 알려진 리터럴(`127.0.0.1`, `::1`, `localhost`)에 더해 `127.0.0.0/8` 전 대역을 포함한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 공개 바인드 금지를 디폴트로 정한 결정 기록 미확인
- [ ] OPEN: 외부 노출 opt-in 환경 변수의 정확한 정책(경고 동반 여부, 문서화 수준) 미확인
