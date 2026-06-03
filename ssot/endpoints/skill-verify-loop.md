---
id: endpoint.skill-verify-loop
kind: Endpoint
title: 스킬 clawket-verify-loop
definition: 검증 라운드 하나를 처음부터 끝까지 진행하는 러너 스킬 — 서브에이전트 배치 디스패치, 7필드 TSV 근거 수집, Clawket 태스크 전사, 3-way 수렴 판정을 거쳐 다음 라운드를 예약하거나 루프를 종료한다.
realizedBy:
  - component.skills
implementedIn:
  - clawket/skills/clawket-verify-loop/SKILL.md
relatesTo:
  - to: concept.scenario
    type: reads
    note: 유닛별 시나리오 지식을 배치로 나눠 검증 대상으로 삼는다
  - to: concept.tsv-evidence
    type: backed-by
    note: 라운드의 모든 판정을 7필드 TSV로 수집한 뒤 태스크로 전사한다
  - to: concept.convergence
    type: reads
    note: 라운드 종료 시 결함/시나리오오류/수렴 3-way 판정을 렌더링한다
  - to: concept.tier
    type: reads
    note: 주의력 희석 의심 시 Opus로 티어를 올리고 에스컬레이션 사유를 기록한다
  - to: concept.task
    type: mutates
    note: 동기화된 QA 태스크와 회귀 조사 태스크를 사이클에 등록한다
governedBy: []
impacts:
  - concept.convergence
  - domain.pdd-verification
consumedBy:
  - persona.qa-reasoner-subagent
  - persona.llm-coding-agent
dependsOn:
  - endpoint.cli-discover-loop
  - endpoint.skill-verify-batch
  - endpoint.skill-defect-fix
  - endpoint.skill-scenario-refine
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

clawket-verify-loop는 활성 플랜·사이클 위에서 검증 라운드 한 회차를 끝까지 돌리는 진입 러너다. 시나리오 작성 스킬에서 의도를, 플랜 설계 스킬에서 플랜·유닛을 받아 시작한다. 라운드는 "시나리오 정제 + 코드 검증"이 함께 진화하는 단위로 모델링된다 — 코드도 시나리오도 신성불가침이 아니며, 의도와 동작이 어긋나면 어느 쪽이든 갱신될 수 있다. 다만 시나리오를 고치는 정당한 사유는 오직 "의도 불일치"뿐이고, 시간·비용·복잡도·코드 영향 크기는 사유로 인정되지 않는다.

한 라운드의 흐름은 라운드 진입 점검 → 배치 디스패치 → TSV 근거 수집 → 벌크 전사 → 3-way 수렴 판정 → 다음 라운드 예약(또는 루프 종료)이다. 결함=0이면서 시나리오오류=0인 상태가 2라운드 연속이면 수렴으로 보고 루프를 닫는다.

## 요청 / 응답

시작 입력은 활성 플랜 식별자, 활성 사이클 식별자(없으면 생성·활성화), 라운드 번호, 그리고 각 유닛의 시나리오 지식 식별자가 담긴 유닛 목록이다.

라운드 종료 시점에 이 스킬은 3-way로 분기한다. 결함이 하나라도 있으면 결함 행을 결함수정 스킬에 넘긴다(수정 태스크가 완료되면 원래 QA 태스크가 자동 해제된다). 시나리오 오류가 하나라도 있으면 그 행을 시나리오 정제 스킬에 넘겨 분할·의도재정의·삭제 중 하나를 결정하고 시나리오 지식과 감사 지식을 갱신한다. 결함과 시나리오 오류가 모두 0이면 수렴 조건을 확인해 충족 시 루프를 닫고(플랜 본문에 수렴 기록을 덧붙임) 아니면 다음 라운드를 예약한다.

회귀 감지도 한다. 이번 라운드의 결함 수가 직전 라운드보다 많으면(상태 응답이 회귀를 보고하면) 곧바로 우선순위 회귀 조사 태스크를 수정 플랜의 해당 라운드 유닛에 등록하고 사용자 확인을 요청한다. 결함 수의 단조 감소가 건강한 패턴이며, 증가는 수정이 새 결함을 들였다는 신호다.

산출물은 라운드의 TSV 근거 지식, 사이클에 동기화된 QA 태스크, 수렴 평결(계속/수렴/차단), 그리고 필요 시 다음 라운드 예약이다.

## 권한 / 제약

배치 크기는 30 시나리오를 넘을 수 없다(초과 시 디스패치 거부, 주의력 희석 방지). 모든 태스크는 시나리오 식별자를 가져야 하며(스키마 NOT NULL), 결함·시나리오 오류 행은 근거(파일:줄)가 없으면 거부된다. 전사 코드 안에는 추론이 들어가서는 안 된다 — 판정은 TSV가 했고 전사는 옮기기만 한다.

훅 정책은 데몬과 에이전트 양쪽에서 강제된다. 데몬의 verify-tsv·dispatch-plan 엔드포인트는 스키마 수준 가드만 하고, 에이전트 측 PostToolUse 훅이 운영 안티패턴을 막는다: 시나리오 식별자가 없는 새 태스크는 취소·코멘트, dispatch-plan 매니페스트가 배치 30 초과를 보고하면 디스패치 중단, 근거 없는 TSV 행은 데몬이 동기화 거부, 전사 핸들러 안의 추론 호출은 코드 리뷰·린트로 차단. 훅 우회는 사용자의 명시적 동의를 요한다.

수렴 판정에서 데몬은 태스크의 qa_status 또는 status를 함께 보고(lock-step) 부분 컬럼 드리프트가 집계를 왜곡하지 않게 한다. 다음 라운드 예약 시 데몬은 직전 플랜의 활성 사이클을 자동 완료하고, 이미 수렴한 플랜에서의 재실행을 막으며(ALREADY_CONVERGED), 추론된 도메인·영역으로 다음 라운드 플랜·사이클·유닛을 만든다.

자율 실행 경계: 사용자 데이터 경로 쓰기 금지, DB DROP·DELETE·TRUNCATE 금지, git reset·commit·push·tag·release 금지, DB 직접 DDL 금지. 비파괴적 컬럼 추가만 허용되며 위반은 사용자 확인을 요한다.

## provenance

이 스킬은 번들 스킬 묶음(component.skills)의 하나로 `clawket/skills/clawket-verify-loop/SKILL.md`에 정의된다. 결정적 상태는 데몬의 discover-loop 명령군(endpoint.cli-discover-loop: dispatch-plan/sync/status/converged/next-round/rounds)이 소유하고, 이 스킬은 LLM 판단 영역(서브에이전트 디스패치, 주의력 희석 자가점검, 시나리오 정제 위임, 플랜 수렴 기록, 다음 라운드 예약)을 소유한다. 배치 운영은 verify-batch에, 결함 처리는 defect-fix에, 시나리오 오류 처리는 scenario-refine에 위임한다.

## 미확정 (OPEN)
- [ ] OPEN: 다음 라운드 예약의 1200초 지연(ScheduleWakeup) 값과 동기 진행(/loop) 사이의 선택 기준이 운영 디폴트로 고정인지 — owner 확인.
