---
id: invariant.evidence-required-on-done
kind: Invariant
title: 태스크 완료에는 근거(evidence)가 반드시 따라야 한다
definition: 태스크를 done 상태로 전환하려면 비어 있지 않은 evidence(file:line 위치 또는 추론 요약)가 반드시 있어야 하며, 없으면 데몬이 완료를 거부한다.
governs:
  - concept.task
  - capability.manage-task-lifecycle
  - domain.work-management
implementedIn:
  - daemon/src/repo/tasks.rs
  - daemon/src/routes/error.rs
decidedBy:
crossesBoundary: true
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

태스크가 "완료(done)" 상태가 되려면, 그 완료를 뒷받침하는 근거가 반드시 함께 기록되어 있어야 한다. 근거는 두 가지 형태 중 하나다 — 변경이 일어난 코드 위치(파일명과 줄 번호), 또는 무엇을 어떻게 끝냈는지에 대한 추론 요약 문장. 근거가 비어 있거나 공백만 있으면 그 태스크는 완료로 넘어갈 수 없다.

이 제약은 단순한 권고가 아니라 시스템이 거절로 강제하는 규칙이다. 사람이든 LLM 에이전트든, 어느 경로로 완료를 시도해도 동일하게 적용된다. 이미 완료 상태인 태스크를 다시 같은 상태로 두는 경우(상태 진입이 아닌 경우)에는 추가 검사를 하지 않으며, 검사는 "다른 상태에서 done 으로 새로 진입할 때"만 작동한다.

## 깨지면 무슨 일이 일어나나

만약 이 규칙이 없다면, "다 했다"는 선언만으로 태스크가 닫히고 무엇을 근거로 끝났다고 판단했는지가 남지 않는다. 나중에 작업을 검토하거나 회귀를 추적할 때 "이게 정말 끝난 게 맞는지", "어디를 바꿔서 끝났는지"를 알 수 없게 되어, 작업 관리가 신뢰할 수 없는 빈 체크박스로 전락한다. 특히 LLM 에이전트가 검증 없이 태스크를 양산-완료하는 것을 막는 안전장치가 사라진다.

규칙이 작동하면, 근거 없는 완료 시도는 즉시 실패로 되돌아오고(권한 거부에 해당하는 오류로), 에이전트나 사람은 근거를 채워 넣은 뒤에야 완료할 수 있다.

## 코드에서 어떻게 강제되나

이 제약은 데몬(로컬 백엔드 서버) 쪽에서 강제된다. 태스크 상태 변경을 처리하는 저장소 계층(`daemon/src/repo/tasks.rs`)에서, done 으로 새로 진입하는 경우에 한해 "이 변경 이후 실제로 남는 근거 값"을 계산한다 — 이번 요청에서 명시적으로 채운 값이 있으면 그 값을, 없으면 기존 태스크에 이미 붙어 있던 값을 본다. 그 값이 공백을 빼면 비어 있으면 `EVIDENCE_REQUIRED` 오류로 거부한다. 이 오류 코드는 응답 변환 계층(`daemon/src/routes/error.rs`)에서 HTTP 400(잘못된 요청)으로 매핑되어 호출자에게 전달된다.

이 검사는 데몬에 있으므로, CLI를 거치든 웹을 거치든 훅을 거치든 단일 지점에서 동일하게 막힌다. 시스템 경계(에이전트/도구 ↔ 데몬)를 가로지르는 계약이다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 규칙이 도입된 결정 기록) 연결 필요 — 코드 주석상 US-CKT-SCHEMA-017 / PDD X8 맥락으로 추정되나 결정 노드 미확정.
