---
id: endpoint.skill-scenario-refine
kind: Endpoint
title: 스킬 clawket-scenario-refine
definition: 검증 라운드가 시나리오 오류 행을 보고할 때, 각 시나리오를 원자 분할·의도 재정의·삭제 셋 중 하나로 처분하고 스펙 지식과 감사 로그를 갱신하는 라운드 내 시나리오 정제 스킬.
realizedBy:
  - component.skills
implementedIn:
  - clawket/skills/clawket-scenario-refine/SKILL.md
relatesTo:
  - to: concept.scenario
    type: mutates
    note: 시나리오를 분할·의도재정의·삭제로 처분하고 스펙 지식을 갱신한다
  - to: concept.knowledge
    type: mutates
    note: 스펙 지식 본문(현재 의도만)과 append-only 감사 지식을 갱신한다
  - to: concept.audit-log
    type: relates-to
    note: 정제 결정마다 감사 지식에 한 행씩 누적 기록한다(취소된 QA 태스크 코멘트와 함께)
  - to: concept.tsv-evidence
    type: reads
    note: 시나리오 오류 행의 scenario_amendment 제안을 입력으로 받는다
governedBy: []
impacts:
  - concept.scenario
  - domain.pdd-verification
consumedBy:
  - endpoint.skill-verify-loop
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

clawket-scenario-refine는 검증 라운드의 TSV에 시나리오 오류 행이 나타날 때 검증 루프 러너가 호출하는 스킬이다. 시나리오 오류란 시나리오 자체가 코드 의도와 모순되거나 한 시나리오에 두 가정이 섞여 있는 등, 코드가 아니라 시나리오를 고쳐야 하는 경우를 말한다. 이 스킬은 각 시나리오를 세 갈래(원자 분할·의도 재정의·삭제) 중 정확히 하나로 처분하고, 그 결정을 스펙 지식·감사 지식·취소된 QA 태스크 코멘트에 기록한다.

## 요청 / 응답

입력은 시나리오 오류로 판정된 시나리오 식별자들, 각 행의 수정 제안(scenario_amendment), 원본 시나리오 지식(스펙 타입), 그리고 도메인별 감사 지식(없으면 생성)이다.

처분은 다음 셋 중 하나다. 원자 분할은 한 시나리오를 N개로 쪼개는 것으로, 현재 최대 번호 다음의 연속 번호로 새 식별자를 발급하고 원본 식별자는 영구히 비운다(재사용 금지). 각 새 시나리오는 자기만의 As-a/I-want/So-that와 하나의 Given/When/Then을 갖는다. 의도 재정의는 식별자를 그대로 두고 본문(역할·바람·이유·Given/When/Then)만 교체하는 것으로, 제품 비전과 어긋난 기대 결과를 바로잡을 때 쓴다. 삭제는 차기 대규모 범위로 미루거나 기능을 드롭할 때로, 식별자를 영구 비운다(다른 곳으로 옮기면 그곳에서 새 식별자 발급).

산출물은 갱신된 스펙 지식(현재 의도만), append-only 감사 지식, 취소된 QA 태스크 코멘트, 그리고 다음 라운드를 위한 검증 루프 러너로의 복귀 신호다. 정제된 시나리오는 다음 라운드에서 새 태스크로 재평가된다.

## 권한 / 제약

시나리오를 고치는 유일한 정당 사유는 "의도 불일치"다. 한 시나리오에 두 가정이 섞임, 기대 결과가 제품 비전과 모순, 시나리오가 보류됨은 수용된다. 반면 시간·비용·복잡도·코드 영향 크기는 거부된다 — 이는 시나리오를 약화시키려는 시도로 간주한다. 시간·비용이 동기인 수정은 정제를 멈추고 사용자 확인을 요청하며, 코드 영향이 크면 영향 받는 코드 경로 전부를 다루는 별도 수정 플랜을 등록한다.

식별자 무결성은 절대적이다: 시나리오 식별자는 한 번 부여되면 절대 재사용되지 않고(번호는 영구), 삭제·분할로 비워진 식별자는 영구히 비운 상태로 남으며(라운드 간 추적성 보존), 원자 분할의 새 식별자는 현재 최대 번호 다음의 연속 번호를 쓴다.

지식 본문에는 변경 이력(changelog)을 절대 남기지 않는다 — 본문은 현재 의도만 담고, 히스토리는 취소된 QA 태스크 코멘트와 감사 지식으로 분리한다. 자율 실행 경계: 런타임·DB DDL·git 작업 금지, 의심스러운 수정은 사용자 확인.

## provenance

이 스킬은 번들 스킬 묶음(component.skills)의 하나로 `clawket/skills/clawket-scenario-refine/SKILL.md`에 정의된다. 검증 루프 러너(endpoint.skill-verify-loop)가 시나리오 오류 수가 0보다 클 때 호출한다. 코멘트·지식 기록은 데몬의 comment·knowledge 경로를 통해 이뤄지며, 처분 결정 자체는 LLM 판단 영역이다.

## 미확정 (OPEN)
- [ ] OPEN: scenario_amendment 필드가 데몬 sync 시 자동 추출되는지(verify-loop의 sync 설명) 또는 정제 스킬이 취소된 태스크 본문에서 직접 읽는지 — 두 경로의 현행 디폴트를 owner 확인.
