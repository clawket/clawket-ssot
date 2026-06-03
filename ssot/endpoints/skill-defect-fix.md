---
id: endpoint.skill-defect-fix
kind: Endpoint
title: 스킬 clawket-defect-fix
definition: 검증 라운드가 결함 행을 보고할 때, 결함마다 결함해결 플랜의 Round R 유닛 아래 수정 태스크를 등록하고 거기서 코드를 고친 뒤(QA 플랜에서는 절대 고치지 않음) Done 명제가 외부에서 검증 가능한지 확인하는 결함 수정 스킬.
realizedBy:
  - component.skills
implementedIn:
  - clawket/skills/clawket-defect-fix/SKILL.md
relatesTo:
  - to: concept.task
    type: mutates
    note: 결함 행 하나당 수정 태스크 하나를 결함해결 플랜의 Round R 유닛에 1:1로 등록한다
  - to: concept.scenario
    type: reads
    note: 의도가 위반된 시나리오 식별자를 수정 태스크에 필수로 단다
  - to: concept.tsv-evidence
    type: reads
    note: 결함 행의 reasoning과 evidence(파일:줄)를 수정 태스크의 근거로 가져온다
  - to: concept.execution-envelope
    type: reads
    note: 수정 태스크에 intent·prompt_template·success_criteria를 부여한다
governedBy: []
impacts:
  - concept.convergence
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

clawket-defect-fix는 검증 라운드의 TSV에 결함 행이 있을 때 검증 루프 러너가 호출하는 스킬이다. 결함이란 시나리오가 코드에 도달하지 못하거나, 도달했지만 결과가 틀리거나, 분기가 누락된 경우 — 즉 코드를 고쳐야 하는 경우다. 이 스킬은 결함마다 수정 태스크를 등록하고 그 안에서만 코드를 고치며, Done 정의가 외부에서 검증 가능한 명제인지 확인한다. 핵심 규율은 코드 변경이 QA 플랜이 아니라 별도의 결함해결 플랜 안에서만 일어난다는 것이다 — QA 플랜은 추론 전용이다.

## 요청 / 응답

입력은 라운드 번호, 결함으로 판정된 TSV 행들(각 행은 시나리오 식별자·reasoning·근거를 가짐), 결함해결 플랜 식별자(없으면 생성), 그 플랜 안의 Round R 유닛 식별자(없으면 생성), 활성 사이클 식별자다.

절차는 다음과 같다. 먼저 결함해결 플랜을 확보한다 — 이 플랜은 도메인당 하나로 모든 라운드가 공유한다(라운드별이 아님). 다음 Round R 유닛을 확보한다(수정 플랜 안 라운드당 하나). 그다음 결함 행마다 수정 태스크를 하나씩 등록하는데, 제목은 단일 동사구로 시작하고(FIX: …), 시나리오 식별자와 근거(파일:줄)를 필수로 달며, 수정 의도·구현 방법·검증 가능한 Done 조건을 부여한다. 원래 QA 태스크에는 수정 태스크 식별자를 코멘트로 연결한다. 그 후 수정 태스크를 진행 중으로 바꿔 코드를 고치고(근거의 파일:줄에서 시작해 필요한 만큼만 바깥으로 추적), 변경 후 다음 라운드에서 이 시나리오가 통과로 판정될지 코드 추론으로 검증한 뒤 수정 위치·내용·Done 검증을 담아 완료한다.

모든 결함의 수정 태스크가 완료되면 다음 라운드 서브에이전트 디스패치를 위해 제어권을 검증 루프 러너에 되돌린다. 다음 라운드는 같은 시나리오를 새 태스크로 재평가한다.

## 권한 / 제약

수정 태스크의 Done 명제는 자기참조여서는 안 된다. "수정 완료"·"구현 완료"는 거부되고, 외부 명제로 다시 써야 한다 — 예: "다음 라운드에서 해당 시나리오가 통과로 판정된다", "변경 후 Given→When→Then 추적이 기대 결과에 깨끗이 도달한다".

다음은 거부되는 안티패턴이다: QA 플랜 안에서의 코드 변경, 추적 수정 태스크 없는 코드 변경(감사 추적 부재), Done="코드 변경 완료"(외부 명제로 재기술 필요), 시나리오 식별자 없는 수정 태스크, 근거 없는 수정 태스크.

자율 실행 경계: 런타임·DB DDL·git 작업 금지. 코드 편집은 수정 태스크가 진행 중일 때만 일어나며(Clawket 훅이 강제), 비파괴적 컬럼 추가만 허용된다. 결함해결 플랜 제목은 "<도메인> QA 이슈 해결"(공유 단일 플랜), 수정 유닛 제목은 "Round R"(라운드당 하나)로 고정된다.

## provenance

이 스킬은 번들 스킬 묶음(component.skills)의 하나로 `clawket/skills/clawket-defect-fix/SKILL.md`에 정의된다. 검증 루프 러너(endpoint.skill-verify-loop)가 결함 행이 있을 때 호출한다. 플랜·유닛·태스크·코멘트 조작은 데몬의 plan·unit·task·comment 경로를 통하며, 수정 태스크의 in_progress 상태에서만 코드 편집을 허용하는 것은 Clawket 훅 강제 영역이다.

## 미확정 (OPEN)
- [ ] OPEN: SKILL.md 본문이 수정 태스크 type을 한 곳은 "bug", 다른 곳(태스크 품질 규칙)은 "code"로 적어 불일치 — 어느 쪽이 현행 규약인지 owner 확인.
