---
id: invariant.db-is-sot-plan-markdown-view
kind: Invariant
title: 플랜의 단일 진실은 DB이고 markdown은 재생성되는 뷰다
definition: "플랜의 단일 진실원천은 Clawket 데이터베이스이며, markdown은 손으로 편집해 진실이 되는 문서가 아니라 DB에서 export로 재생성되는 읽기용 뷰다. Claude의 Plan Mode 로컬 파일도 진실이 아니다."
governs:
  - concept.plan
  - component.sqlite-storage
  - endpoint.cli-plan
  - endpoint.daemon-plans-import-strict
  - endpoint.hook-plan-sync
  - domain.work-management
governs_note: "Plan Mode 출력은 strict 검증을 거쳐 DB에 플랜으로 등록되어야 진실이 되며, 그 전까지의 로컬 파일은 stale 가능한 초안이다."
implementedIn:
  - clawket/README.md
  - clawket/bin/clawket
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

플랜에 관한 "진짜"는 언제나 Clawket 데이터베이스 안에 있다. markdown 형태의 플랜 문서는 사람이 손으로 고쳐서 그것이 곧 진실이 되는 원본이 아니라, DB의 현재 상태를 내보내(export) 만들어내는 읽기용 표현(뷰)이다. 따라서 markdown을 직접 편집하는 것은 진실을 바꾸는 행위가 아니며, 플랜을 바꾸려면 DB를 통해(CLI·데몬) 바꿔야 한다.

마찬가지로, Claude의 Plan Mode가 남기는 로컬 플랜 파일도 진실이 아니다. 그 파일들은 시간이 지나면 실제 상태와 어긋날 수 있는(stale) 초안일 뿐이다. 에이전트는 대화 중에 플랜을 제안하고, 사용자가 승인하면 그것을 CLI를 통해 DB에 등록한다 — 등록된 그 순간부터 비로소 DB가 그 플랜의 진실이 된다.

## 깨지면 무슨 일이 일어나나

이 제약이 흐려지면 "어느 버전이 맞는 플랜인가"가 모호해진다. 누군가 markdown 파일을 손으로 고쳐놓고 그게 반영된 줄 알면, DB의 실제 상태와 문서가 갈라져(divergence) 사람과 에이전트가 서로 다른 플랜을 보고 일하게 된다. Plan Mode의 로컬 파일을 진실로 취급하면 같은 문제가 생긴다 — 그 파일은 등록되지 않는 한 시스템 상태와 동기화되지 않으므로 오래된 정보를 진실로 오인하게 된다. 단일 진실원천(single source of truth) 원칙이 이 모호함을 막기 위한 것이다.

## 코드에서 어떻게 강제되나

이 불변식은 정책(README.md)에 "Clawket이 플랜의 단일 진실원천이며 Plan Mode 파일이 아니다"로 명문화되어 있고, 동작으로도 뒷받침된다. 플랜 명령은 markdown을 DB에서 내보내는(export) 방향으로 제공되므로 markdown은 항상 DB로부터 재생성되는 산출물이다. 또한 Plan Mode 종료를 가로채는 동기화 경로는, 그 출력을 strict 형상 검증을 거쳐 Clawket 플랜으로 등록하도록 유도한다 — 즉 로컬 초안이 진실이 되려면 반드시 DB 등록이라는 관문을 통과해야 한다. 다만 markdown 파일에 대한 손편집 자체를 코드가 실시간으로 막지는 않으며, "그 편집은 진실이 아니다"라는 규약으로 무력화된다(export가 덮어쓴다).

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(DB를 플랜 단일 진실원천으로 정한 근거 결정) 연결 필요
