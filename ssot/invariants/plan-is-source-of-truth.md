---
id: invariant.plan-is-source-of-truth
kind: Invariant
title: 플랜의 정본은 Clawket DB이지 로컬 Plan Mode 파일이 아니다
definition: 플랜의 단일 진실은 Clawket 데이터베이스에 저장된 플랜 본문이다. Claude Code의 Plan Mode가 로컬에 남기는 파일은 시간이 지나면 stale해지므로 정본으로 신뢰하지 않으며, 별도의 markdown은 검토 편의용 사본일 뿐이다.
governs:
  - concept.plan
  - endpoint.hook-plan-sync
  - endpoint.daemon-plans-import-strict
  - capability.plan-and-approve
implementedIn:
  - clawket/adapters/claude/plan-sync.cjs
  - clawket/hooks/hooks.json
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

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 제약

"이 작업의 계획이 무엇인가"라는 질문에 대한 정답은 항상 Clawket 데이터베이스 안에 등록된 플랜 본문이다. Claude Code가 계획 모드(Plan Mode)에서 만들어 로컬 디렉터리에 남기는 계획 파일은 정본이 아니다. 그 파일은 한 번 만들어진 뒤 작업이 진행되며 내용이 바뀌어도 갱신되지 않아 곧 현실과 어긋나기(stale) 때문이다. 따로 내보낸 markdown 문서가 있더라도 그것은 사람이 읽기 위한 사본일 뿐, 진실의 출처가 아니다.

따라서 계획이 바뀌면 로컬 파일이 아니라 Clawket에 등록된 플랜을 갱신해야 하고, 무엇이 진짜 계획인지 확인할 때도 Clawket의 플랜을 본다.

## 깨지면 무슨 일이 일어나나

이 원칙이 지켜지지 않고 로컬 Plan Mode 파일을 정본처럼 신뢰하면, 에이전트와 사람이 서로 다른(낡은) 계획을 보고 작업하게 된다. 태스크는 Clawket의 플랜·사이클 체계 안에서만 시작될 수 있으므로, 로컬 파일에만 있는 계획은 실제 작업 흐름과 연결되지 않은 채 표류한다. 결과적으로 "승인된 의도"와 "실제로 추적되는 작업" 사이에 균열이 생기고, 대시보드의 로드맵이 현실을 반영하지 못한다.

## 코드에서 어떻게 강제되나

Claude Code의 계획 모드 종료(ExitPlanMode) 시점에 훅이 그 출력을 가로채, 엄격한 형상 검증을 거쳐 Clawket의 플랜으로 등록하도록 유도한다. 이렇게 함으로써 계획이 로컬 파일에만 머무는 것을 막고 데이터베이스로 끌어와 정본화한다. 등록 이후 플랜은 승인(approve)을 거쳐야 비로소 태스크 시작 자격(active)을 얻으므로, 작업 흐름 전체가 데이터베이스의 플랜을 기준으로 돌아간다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 불변식을 도입한 결정 근거) 연결 필요
