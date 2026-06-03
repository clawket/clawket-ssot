---
id: invariant.path-separation
kind: Invariant
title: 데이터 경로 분리(플러그인 디렉터리와 격리, LM-8)
definition: Clawket의 데이터·캐시·설정·상태(특히 SQLite DB)는 Claude Code 플러그인 디렉터리(~/.claude/plugins/) 아래에 두어서는 안 된다. 플러그인 재설치가 그 트리를 통째로 지우기 때문이다. doctor가 이 위반을 진단한다.
governs:
  - component.sqlite-storage
  - component.daemon
  - platform.local-first
governedBy: []
implementedIn:
  - cli/src/paths.rs
  - cli/src/doctor.rs
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

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 제약
Clawket이 영속 데이터를 보관하는 모든 경로 — 데이터, 캐시, 설정, 상태, 그리고 그 안의 SQLite 데이터베이스 — 는 Claude Code의 플러그인 설치 디렉터리(`~/.claude/plugins/`) 안에 위치해서는 안 된다. 이 경로 분리 불변식은 내부적으로 LM-8로 명명되어 있다.

이유는 단순하고 치명적이다. Claude Code에서 플러그인을 재설치(`/plugin install`)하면 그 플러그인 트리가 통째로 지워진다. 만약 Clawket의 DB가 그 안에 있었다면, 다음 플러그인 업데이트 한 번에 모든 작업 상태·지식·감사 로그가 조용히 소멸한다. 따라서 데이터 경로는 항상 플러그인 트리 바깥(예: 사용자 데이터/캐시/설정 표준 위치)을 가리켜야 한다.

## 깨지면 무슨 일이 일어나나
DB나 데이터 경로가 플러그인 디렉터리 아래로 해석되면, 플러그인 재설치/업데이트 시점에 단일 진실 저장소가 흔적 없이 파괴된다. 사용자는 자신이 한 행동(플러그인 업데이트)이 데이터 소멸의 원인임을 인지하기 어렵다 — 그래서 doctor가 이를 사전에 빨간불로 잡아낸다. 위험을 알고도 감수하려면 별도 환경변수로 명시적으로 승인해야 한다.

## 코드에서 어떻게 강제되나
경로가 플러그인 디렉터리와 겹치는지 판정하는 술어가 CLI에 있다(`cli/src/paths.rs`의 `path_overlaps_plugin_dir` — 경로 문자열에 `/.claude/plugins/`가 포함되는지 검사). 런타임 가드는 데몬이 소유하고, CLI는 동일 상태를 **보고**한다. `clawket doctor`가 데이터·캐시·설정·상태·DB 다섯 경로를 각각 검사해 하나라도 겹치면 오류로 표시하고, 비정상 종료로 끝난다(`cli/src/doctor.rs`의 "Path separation invariant" 섹션). 명시적 환경변수로 위험을 승인하면 통과시킨다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(LM-8 경로 분리 결정의 근거) 연결 필요
- [ ] OPEN: 데몬 측 런타임 가드 구현 위치 확인 필요
