---
id: invariant.evidence-on-done
kind: Invariant
title: 태스크 완료(done)에는 근거(evidence)가 필수
definition: 태스크를 완료(done) 상태로 전환하려면 비어 있지 않은 근거(file:line 또는 추론 요약)가 반드시 있어야 하며, 없으면 데몬이 완료를 거부한다.
governs:
  - concept.task
  - domain.work-management
implementedIn:
  - daemon/src/repo/tasks.rs
  - cli/src/main.rs
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

<!-- 작성 고도: 자연어 우선. 식별자는 provenance/근거에만. -->

## 제약

태스크를 완료로 표시하는 순간에는 "왜 끝났다고 보는가"를 설명하는 근거가 반드시 함께 있어야 한다. 근거는 코드 위치(파일:줄) 형태이거나 추론 요약 형태일 수 있으며, 공백뿐인 값은 근거 없음으로 취급한다. 이 검사는 "완료로 들어가는" 전환에만 적용된다 — 이미 완료된 태스크에 대한 반복 처리는 새 근거를 요구하지 않는다(멱등). 근거는 이번 완료 요청에서 새로 제시하거나, 태스크에 이미 저장돼 있던 값을 그대로 인정한다. 단, 완료로 전환하면서 근거를 명시적으로 비우는 요청은 거부된다.

## 깨지면 무슨 일이 일어나나

근거 없이 태스크가 완료될 수 있다면, "완료됨"이라는 상태가 검증되지 않은 자기 선언으로 전락한다. 사람이나 다른 에이전트가 나중에 그 작업이 실제로 무엇을 근거로 끝났는지 추적할 수 없고, 검증 루프와 감사 추적이 근거를 잃는다. 특히 LLM 에이전트가 작업을 자동 완료할 때 근거 강제가 없으면, 실제로는 미완인 작업이 done으로 쌓여 작업 상태 전체의 신뢰가 무너진다.

## 코드에서 어떻게 강제되나

데몬의 태스크 상태 전환 로직에서 완료로 진입할 때 유효 근거를 계산해, 비어 있으면 거부한다(`daemon/src/repo/tasks.rs`, EVIDENCE_REQUIRED — v3에서 soft-warn이 아니라 하드 400으로 강화됨). 유효 근거는 이번 패치에 담긴 값 또는(패치에 없으면) 기존 저장 값으로 결정되며, 완료 전환과 함께 근거를 null로 비우는 패치도 실패시킨다. CLI 완료 명령(`cli/src/main.rs`)도 근거 인자를 받아 이 규칙을 표면화한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — done 근거 강제를 soft-warn에서 하드 400으로 올린 결정(US-CKT-SCHEMA-017 / PDD X8)을 decision 노드로 연결.
