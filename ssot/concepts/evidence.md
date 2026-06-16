---
id: concept.evidence
kind: Concept
title: 근거(Evidence)
definition: "태스크를 완료(done)로 전환할 때 반드시 첨부해야 하는 증빙. 코드의 file:line 참조이거나 추론 요약(free-text)이며, 작업 도구(데몬)가 done 전환 시 그 존재를 강제한다(EVIDENCE_REQUIRED)."
relatesTo:
  - { to: concept.task, type: belongs-to, note: "태스크의 done 전환에 첨부되는 증빙." }
  - { to: concept.tsv-evidence, type: reads, note: "검증 라운드의 TSV 근거 중 evidence(file:line) 필드가 이 개념의 행 단위 형태다." }
  - { to: concept.run, type: reads, note: "서브에이전트 자동 완료 시 결과 요약이 근거로 함께 기록된다." }
governedBy:
  - invariant.evidence-required-on-done
  - invariant.evidence-required
implementedIn:
  - cli/src/main.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

## 정의
근거(evidence)는 "이 태스크는 정말로 끝났다"를 뒷받침하는 증빙이다. 두 형태 중 하나를 취한다.

- **코드 참조**: `파일경로:라인` 형식의 위치 지목.
- **추론 요약**: 무엇을 했고 왜 완료로 볼 수 있는지를 적은 자유 텍스트.

근거의 본질은 강제성이다. 태스크를 완료 상태로 옮기는 행위에는 반드시 근거가 동반되어야 하며, 근거 없이 완료를 시도하면 작업 도구가 거부한다. 즉 "끝났다고 선언만 하고 흔적을 남기지 않는" 완료를 구조적으로 막는 장치다.

이는 화면(데스크탑·웹)에서도 별도 칩(Evidence) UI로 노출되어, 사람 운영자가 어떤 근거로 태스크가 닫혔는지 한눈에 볼 수 있게 한다.

## 엔티티 (DB)
근거는 태스크의 완료 전환에 결합된 값으로 보존된다. 완료(done) 또는 취소가 아닌 일반 완료 경로에서 근거 문자열이 태스크/실행 기록에 남는다. 서브에이전트가 성공으로 자동 완료될 때는 그 결과 요약이 근거로 기록된다.

## API 표면
- 태스크 완료 단축 명령은 근거를 필수 인자로 받는다. 데몬이 근거 없는 done을 거부하므로 이 입력은 생략 불가다.
- 태스크 생성/갱신 입력에도 근거 필드가 있으며, done 전환 시점에 이 값이 채워져 있어야 한다.
- 데스크탑/웹 데이터 계층과 EvidenceChip 컴포넌트가 이 값을 읽어 표시한다.

## 불변식
- 태스크를 done으로 전환하려면 근거가 있어야 한다(invariant.evidence-required-on-done / invariant.evidence-required). 근거가 없으면 데몬이 EVIDENCE_REQUIRED로 거부한다.

## 구현 위치 (provenance)
- CLI 태스크 생성/완료 명령 정의에서 evidence 옵션이 "file:line 참조 또는 추론 요약, done 전환 시 필수(EVIDENCE_REQUIRED)"로 선언되어 있고, 완료 단축 명령에서는 필수 인자로 강제된다(`cli/src/main.rs`의 태스크 evidence 옵션 및 Complete 정의 부근).
- 화면 표시는 데스크탑/웹 데이터 API와 EvidenceChip UI 컴포넌트가 담당한다(인벤토리 소스 기록).

## 미확정 (OPEN)
- [ ] OPEN: 취소(cancelled) 전환에도 근거가 요구되는지(완료 경로 전용 강제인지) 코드 추가 확인 필요.
