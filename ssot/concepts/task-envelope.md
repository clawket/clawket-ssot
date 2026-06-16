---
id: concept.task-envelope
kind: Concept
title: 태스크 실행 계약(intent · prompt_template · success_criteria)
definition: "태스크를 실행하기 위해 반드시 채워져 있어야 하는 최소 실행 계약. 태스크가 '무엇을 달성하는지(intent)', '실행 에이전트가 돌릴 프롬프트 본문(prompt_template)', '완료로 인정되는 기준(success_criteria)'을 묶은 것으로, 부모 태스크 체인을 따라 상속·해석되며 데몬이 누락을 거부한다."
relatesTo:
  - { to: concept.execution-envelope, type: belongs-to, note: "이 3필드는 더 큰 19필드 실행 봉투(execution-envelope)의 핵심 부분집합이다." }
  - { to: concept.task, type: belongs-to, note: "각 태스크에 서명되는 실행 계약." }
  - { to: concept.scenario, type: reads, note: "검증 흐름에서 태스크는 시나리오에 1:1로 매핑되며 그 의도가 intent로 들어간다." }
governedBy:
  - invariant.envelope-required-fields
  - invariant.secret-never-in-envelope
implementedIn:
  - cli/src/main.rs
  - cli/src/mcp.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

## 정의
태스크 실행 계약은 한 태스크를 실행에 부치기 전에 반드시 합의되어 있어야 하는 세 가지 약속이다.

- **intent**: 이 태스크가 무엇을 달성하는지를 한 문장으로 진술한 것.
- **prompt_template**: 실행을 맡은 에이전트가 실제로 돌리게 될 프롬프트 본문.
- **success_criteria**: 무엇을 만족해야 "완료"로 인정되는지의 판정 기준. 기준은 여러 개일 수 있다.

핵심 성격은 두 가지다. 첫째, 이 계약은 **부모 태스크 체인을 따라 위로 거슬러 해석(resolve)된다** — 자식 태스크가 직접 값을 갖지 않더라도, 활성 봉투를 가진 부모 태스크에 연결되어 있으면 그 부모로부터 계약을 물려받는다. 둘째, 이 계약은 **선택이 아니라 강제**다. 해석된 체인에 위 필드가 비어 있으면 작업 도구(데몬)가 태스크 생성·실행을 거부한다.

이 세 필드는 더 큰 "실행 봉투(execution-envelope)"의 핵심 부분집합으로, 봉투가 강제하는 "분해 → 계약 → 실행" 루프의 계약(contract) 단계를 담당한다.

## 엔티티 (DB)
태스크 실행 계약은 독립된 테이블이 아니라 태스크에 부속된 실행 봉투의 일부로 저장된다. 데이터 모델 관점에서 한 태스크는 자신의 intent·prompt_template·success_criteria를 직접 보유하거나, 부모 태스크가 보유한 활성 봉투를 상속한다. 비밀(시크릿)은 이 계약 어디에도 평문으로 들어가서는 안 된다.

(봉투 전체의 저장 구조는 concept.execution-envelope 문서를 단일 원본으로 본다.)

## API 표면
태스크를 생성할 때 이 계약은 별도 입력으로 함께 채워진다. 생성 입력에는 의도(intent), 실행 에이전트가 돌릴 프롬프트 본문(prompt_template), 완료 기준(success_criteria, 반복/콤마 구분 가능)이 포함된다. 해석된 부모 체인이 이미 활성 계약을 제공하면 자식 생성 시 이 값들을 생략할 수 있다.

해석된 체인에 필수 필드가 빠져 있으면 작업 도구는 태스크 생성/실행 요청을 잘못된 요청으로 거부한다(필수 봉투 필드 누락).

## 불변식
- 실행에 부쳐지는 모든 리프 태스크는 해석된 체인 어딘가에서 intent·prompt_template·success_criteria를 갖추어야 한다(invariant.envelope-required-fields). 누락 시 데몬이 거부한다.
- 비밀/자격증명은 이 계약(프롬프트 본문·기준·의도)에 평문으로 들어가서는 안 된다(invariant.secret-never-in-envelope).

## 구현 위치 (provenance)
- CLI의 태스크 생성 명령 정의에서 intent·prompt_template·success_criteria 옵션이 봉투 필드로 선언되고, 부모 태스크로부터의 상속 가능성과 데몬의 필수 필드 거부가 함께 문서화되어 있다(`cli/src/main.rs`의 태스크 생성 옵션 정의 부근).
- MCP 측에서는 v3.0에서 봉투 검증·실행 도구가 제거되고 CLI 서브커맨드로 일원화되었음이 기록되어 있다(`cli/src/mcp.rs`).

## 미확정 (OPEN)
- [ ] OPEN: 부모 체인 상속 시 "활성 봉투" 판정의 정확한 조건(어떤 상태의 부모가 상속 가능 봉투를 제공하는지)은 코드 추가 추적 필요.
