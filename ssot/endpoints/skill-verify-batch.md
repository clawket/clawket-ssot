---
id: endpoint.skill-verify-batch
kind: Endpoint
title: 스킬 clawket-verify-batch
definition: 검증 라운드에서 서브에이전트 배치를 디스패치해 시나리오를 코드에 대조시키고, 배치별 7필드 TSV 근거를 만든 뒤, 그 TSV를 16-worker 병렬로 Clawket 태스크에 전사(sync)하는 운영 스킬.
realizedBy:
  - component.skills
implementedIn:
  - clawket/skills/clawket-verify-batch/SKILL.md
relatesTo:
  - to: concept.scenario
    type: reads
    note: 배치 단위로 시나리오를 읽어 코드에 대조한다(배치당 30개 이하)
  - to: concept.tsv-evidence
    type: backed-by
    note: 추론 단계의 산출물이자 전사 단계의 입력인 7필드 TSV 근거를 생성한다
  - to: concept.tier
    type: reads
    note: 기본 Sonnet, 모호하면 Opus로 에스컬레이션하는 티어 라우팅을 적용한다
  - to: concept.task
    type: mutates
    note: TSV 행 하나당 Clawket 태스크 하나를 1:1로 생성하고 상태를 갱신한다
governedBy: []
impacts:
  - concept.convergence
consumedBy:
  - endpoint.skill-verify-loop
  - persona.qa-reasoner-subagent
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

clawket-verify-batch는 검증 라운드의 "디스패치 + 전사" 운영을 담당하는 스킬이다. 상위 스킬인 clawket-verify-loop이 한 라운드를 진행하다가 실제 배치 처리가 필요해지면 이 스킬에 위임한다. 스킬이 하는 일은 두 단계로 엄격히 갈린다. 먼저 시나리오 묶음을 여러 배치로 쪼개 각 배치마다 서브에이전트 하나를 띄워 코드에 비추어 판정하게 하고(추론 단계), 그다음 그 판정 결과를 담은 TSV를 그대로 Clawket 태스크 데이터로 옮겨 적는다(전사 단계).

두 단계를 섞지 않는 것이 이 스킬의 핵심 원칙이다. 전사 단계는 추론을 일절 하지 않는다 — TSV가 이미 결정한 판정을 글자 그대로 옮길 뿐이다. 두 단계가 섞이면 "무엇을 누가 언제 판정했는가"라는 감사 추적이 무너진다.

## 요청 / 응답

이 스킬을 시작하려면 다음이 필요하다: 라운드 번호, 활성 사이클 식별자, 그리고 각 유닛마다 그 유닛이 다룰 시나리오 묶음(지식 항목)과 검증 대상 코드 경로의 목록이다. 배치 크기는 기본 30이며 30을 넘을 수 없다.

추론 단계에서 각 서브에이전트는 시나리오 한 건씩을 코드에 대조한다. "어떤 상태인가(Given)"를 코드의 상태·속성·라우트 파라미터에 매핑하고, "무엇이 일어나는가(When)"를 함수·핸들러·이벤트에 매핑하며, "결과는 어떠해야 하는가(Then)"를 그 함수의 실제 반환값·부수효과·렌더 도달 가능성에 비춘다. 도달하고 결과가 맞으면 통과, 도달 불가·결과 불일치·분기 누락이면 결함, 시나리오 자체가 코드 의도와 모순되면 시나리오 오류로 판정한다. 판정의 핵심 위치(파일:줄)를 근거 필드에 남긴다.

이 스킬이 내놓는 결과물은 세 가지다: 유닛·라운드별 TSV 근거 지식 항목, TSV 행과 1:1로 대응하는 Clawket 태스크 행, 그리고 의심되는 배치 코호트를 격리하기 위한 배치 식별자 추적이다. 처리가 끝나면 수렴 판정을 위해 제어권을 clawket-verify-loop에 되돌려준다.

## 권한 / 제약

배치 분할의 상한은 30 시나리오다. 한 배치가 30을 넘으면 주의력 희석(뒷부분 시나리오의 판정 신뢰도 저하)이 나타나므로 디스패치 자체가 거부된다. 늦은 배치(위치 70% 이상)의 신뢰도가 이른 배치(30% 미만)보다 눈에 띄게 나쁘면 주의력 분산을 의심해 같은 배치의 후반부만 다시 디스패치하거나 티어를 한 단계 올린다.

티어 라우팅은 기본 Sonnet(시나리오-코드 추론의 80~90%)이고, 모호한 경우(시나리오 오류 후보, 경계 불명확)나 회귀 라운드의 결함 근본원인 분석은 Opus로 올린다. Opus를 쓴 모든 행은 에스컬레이션 사유를 반드시 채운다.

전사 단계에는 다음이 금지된다: 전사 핸들러 안에서 서브에이전트를 호출하는 것(추론이 전사에 끼어드는 것), 상태 매핑을 if/else로 분기하는 것(판정은 이미 TSV가 했다), 전사 중 행의 상태를 바꾸는 것이다. 허용되는 것은 순수 문자열 매핑(통과→완료, 결함→차단됨, 시나리오 오류→취소됨)과 DB 쓰기 실패 시 재시도뿐이다(전사는 멱등이다). 전사 워커는 16개를 넘지 않는다. 라운드 종료 시 태스크 행 수는 전체 TSV 행 수와 같아야 하고(1:1), 모든 태스크는 시나리오 식별자·근거·배치 식별자를 갖는다.

자율 실행 경계: 런타임 변경·DB DDL·git 작업은 하지 않는다. 단 비파괴적 컬럼 추가는 허용된다. TSV 근거는 evidence 타입 지식으로 보존하며, 지식 본문은 현재 라운드 내용만 담고 히스토리는 취소된 태스크 코멘트와 감사 지식으로 분리한다.

## provenance

이 스킬은 플러그인이 등록하는 번들 스킬 묶음(component.skills)의 하나로, `clawket/skills/clawket-verify-batch/SKILL.md`에 정의된다. 디스패치·전사의 실제 데이터 흐름은 검증 루프 러너(endpoint.skill-verify-loop)가 호출하고, 전사 자체는 보통 데몬의 discover-loop sync 경로를 통해 이루어진다. 추론 주체는 QA 추론 서브에이전트(persona.qa-reasoner-subagent)다.

## 미확정 (OPEN)
- [ ] OPEN: SKILL.md의 16-worker ThreadPoolExecutor·subprocess 직접 호출 예시가 권장 경로(데몬 discover-loop sync 단일 호출)와 어느 쪽이 현행 디폴트인지 — 두 경로의 우선순위·선택 기준을 owner 확인.
