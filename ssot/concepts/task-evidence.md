---
id: concept.task-evidence
kind: Concept
title: 태스크 근거(evidence)
definition: "태스크 완료의 근거로 태스크에 부착되는 자유 문자열로, file:line 패턴(^[\\w./-]+:\\d+$)에 맞으면 웹 화면이 소스 참조 링크로, 아니면 일반 텍스트로 렌더링한다."
relatesTo:
  - to: concept.task
    type: belongs-to
    note: 근거는 태스크의 PDD v3.0 메타데이터 한 필드로, 태스크 상세에 표시된다.
  - to: concept.tsv-evidence
    type: relates-to
    note: 검증 라운드의 7필드 TSV 근거 중 evidence(file:line) 항목이 태스크 근거 필드로 흘러든다.
  - to: concept.scenario
    type: relates-to
    note: 근거는 같은 PDD 메타데이터 묶음의 scenario_id 와 짝을 이뤄 시나리오 추적성을 이룬다.
governedBy: []
governs: []
dependsOn:
  - concept.task
implementedIn:
  - web/src/types.ts
  - web/src/components/TaskDetail.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 자연어. 코드 옮겨적기 금지. 분기/의도 불명은 OPEN. -->

## 정의

태스크 근거(evidence)는 한 태스크가 **무엇으로 완료를 입증하는가**를 담는 자유 문자열이다. PDD v3.0 의 태스크 추적성 메타데이터 묶음에 속하며, 같은 묶음의 시나리오 ID(scenario_id)·배치 ID(batch_id)와 함께 태스크 상세 화면에 항상 자리한다(값이 없으면 대시 '—' 로 표시).

근거의 **렌더링은 값 모양에 따라 두 갈래로 갈린다**:

- 값이 `파일경로:줄번호` 패턴(예: `src/foo.ts:42`)에 정확히 맞으면, 화면은 이를 **소스 참조 링크**로 그린다(고정폭 글꼴, 클릭 가능, 소스 위치를 툴팁으로 표시). 이 패턴은 검증 라운드의 근거 형식과 같은 모양이라, 코드의 어느 줄이 태스크를 입증하는지로 곧장 연결된다.
- 그 외 임의 텍스트면 일반 텍스트(고정폭, 줄바꿈 허용)로 그린다.
- 값이 없으면(null) 대시로 표시한다.

이 필드는 형식을 강제하지 않는 자유 문자열이지만, file:line 형식을 쓰면 추적성 가치가 가장 높아진다 — 검증 루프가 산출하는 TSV 근거의 evidence 항목이 같은 형식을 쓰며, 그 값이 태스크 근거 필드로 흘러든다.

## 엔티티 (DB)

근거는 태스크 레코드의 한 선택적 문자열 필드다(스키마 v3.0 이전의 옛 레코드는 null). 같은 PDD v3.0 묶음에 시나리오 ID와 배치 ID가 함께 있으며, 셋 다 옛 레코드에서는 null 일 수 있다. 데몬이 done 전환에 근거를 요구하는 규칙은 데몬·CLI 측 책임이며, 웹은 그 값을 표시·링크화한다.

## API 표면

웹은 별도 근거 전용 엔드포인트를 두지 않는다 — 근거는 태스크 레코드의 한 필드로 함께 조회·갱신된다(태스크 조회/수정 표면). 표시 시 file:line 판별과 링크화는 전적으로 화면 로직이다.

## 불변식

- 근거는 형식이 강제되지 않는 **자유 문자열**이다. file:line 은 권장 형식일 뿐 강제는 아니다.
- 화면은 file:line 패턴에 정확히 맞을 때만 링크로, 그 외에는 텍스트로, 없으면 대시로 — 세 갈래 표시가 고정 규칙이다.
- 근거는 태스크에 종속된 한 필드이며 독립 엔티티가 아니다.

## 구현 위치 (provenance)

- `web/src/types.ts` — 태스크의 근거 필드 선언과 "file:line 패턴이면 소스 참조 링크로 렌더링" 의도 주석, PDD v3.0 추적성 묶음(scenario_id/evidence/batch_id).
- `web/src/components/TaskDetail.tsx` — file:line 정규식 판별 후 링크/텍스트/대시 세 갈래로 그리는 근거 표시 로직과 PDD 메타데이터 표시 영역.

## 미확정 (OPEN)
- [ ] OPEN: done 전환 시 근거 필수 여부의 데몬 측 강제 규칙은 데몬 소스 기준 확인 필요.
