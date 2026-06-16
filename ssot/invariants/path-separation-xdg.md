---
id: invariant.path-separation-xdg
kind: Invariant
title: 데이터·캐시·설정·로그 경로를 XDG 규칙으로 분리한다
definition: "DB는 데이터 디렉터리(~/.local/share), 런타임 상태는 캐시 디렉터리(~/.cache), 설정은 설정 디렉터리(~/.config), 로그는 상태 디렉터리(~/.local/state)로 XDG 규칙에 따라 분리되며, 각 경로는 환경변수로 재정의할 수 있고 플러그인 재설치가 사용자 데이터를 건드리지 않는다."
governs:
  - platform.local-first
  - component.sqlite-storage
  - component.install-gate
  - component.daemon
  - component.plugin-shell
governs_note: "경로 분리 덕분에 플러그인 셸(재설치로 교체·재생성되는 영역)과 사용자 데이터(영속 영역)가 다른 디렉터리에 살아, 재설치가 데이터를 지우지 않는다."
implementedIn:
  - clawket/PRIVACY.md
  - clawket/README.md
  - clawket/adapters/shared/claude-hooks.cjs
decidedBy: []
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

Clawket이 기기에 남기는 파일들은 용도에 따라 네 곳으로 나뉘어 살며, 이 분리는 운영체제의 표준 디렉터리 규칙(XDG Base Directory)을 따른다.

- **데이터** — 영구 보존해야 하는 단일 진실(SQLite DB)은 데이터 디렉터리(기본 `~/.local/share/clawket`)에 둔다.
- **캐시·런타임 상태** — 다시 만들 수 있는 임시 산출물과 런타임 상태(예: 데몬 락 파일)는 캐시 디렉터리(기본 `~/.cache/clawket`)에 둔다.
- **설정** — 사용자 설정 값은 설정 디렉터리(기본 `~/.config/clawket`)에 둔다.
- **로그·감사 추적** — 로그와 안티패턴 차단 기록 같은 추적 기록은 상태 디렉터리(기본 `~/.local/state/clawket`)에 둔다. 이 추적 기록은 캐시가 비워져도 살아남아야 하므로 캐시가 아닌 상태 디렉터리에 둔다.

네 경로 모두 환경변수로 위치를 재정의할 수 있어, 사용자가 원하는 디스크 레이아웃에 맞출 수 있다.

이 분리가 지키는 핵심 효과는 **플러그인 재설치가 사용자 데이터를 건드리지 않는다**는 것이다. 재설치·업데이트로 교체·재생성되는 것은 플러그인 셸과 캐시 영역이며, 영속 데이터(DB)는 별도 디렉터리에 있어 영향을 받지 않는다.

## 깨지면 무슨 일이 일어나나

경로 분리가 무너지면 두 가지가 위험해진다. 첫째, 재설치나 캐시 정리 같은 일상적 동작이 사용자의 영속 데이터를 같이 지워버릴 수 있다 — 예컨대 DB가 캐시 디렉터리에 섞여 있으면 캐시 비우기 한 번에 모든 작업 이력이 사라진다. 둘째, 안티패턴 차단·감사 추적 로그가 캐시에 있으면 캐시 만료와 함께 증발해, 무엇이 언제 차단됐는지를 사후에 확인할 수 없게 된다. 이 때문에 추적성이 필요한 로그는 의도적으로 상태 디렉터리에 둔다.

## 코드에서 어떻게 강제되나

경로 분리는 정책 문서(PRIVACY.md·README.md)에 기본 경로와 재정의 환경변수가 명문화되어 있고, 훅 핸들러가 실제 파일을 쓸 때마다 이 규칙을 따라 경로를 해석한다. 핸들러는 로그·락 파일 등을 기록할 때 해당 XDG 환경변수가 있으면 그 아래의 `clawket` 디렉터리를, 없으면 표준 기본 경로를 골라 쓴다. 즉 각 파일이 "어느 용도 디렉터리에 속하는가"가 코드의 경로 해석 로직에 박혀 있다. 이것이 재설치 안전성(플러그인 셸·캐시는 교체 대상, 데이터는 보존 대상)을 떠받친다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(XDG 경로 분리·재설치 안전성 정책의 근거 결정) 연결 필요
