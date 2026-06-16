---
id: concept.evidence-required
kind: Concept
title: 증거 필수(Evidence Required) — 히어로 데모의 핵심 순간
definition: "랜딩 페이지 히어로 데모가 보여주는 단 하나의 핵심 순간 — 태스크를 'done'으로 옮길 때 증거(evidence)를 함께 제출하지 않으면 제출이 거부되는 제품 약속. 화면에서는 제출 버튼이 비활성으로 막히고, 백엔드에서는 같은 시도가 거부(HTTP 400 EVIDENCE_REQUIRED)된다."
relatesTo:
  - { to: invariant.evidence-required-on-done, type: relates-to, note: "이 순간은 'done 전환에는 증거가 필수'라는 불변식을 시각적으로 증명한다" }
  - { to: concept.task, type: reads, note: "증거를 요구받는 대상은 태스크의 done 전환이다" }
  - { to: concept.demo-seed-chain, type: depends-on, note: "이 climax를 찍으려면 in_progress 태스크가 미리 시딩돼 있어야 한다" }
  - { to: screen.home, type: relates-to, note: "히어로 영상으로 랜딩 첫 화면에 노출된다" }
  - { to: screen.web-board, type: reads, note: "데모 영상은 보드의 상태 변경 모달에서 이 순간을 촬영한다" }
governedBy:
  - invariant.evidence-required-on-done
dependsOn:
  - concept.demo-seed-chain
implementedIn:
  - landing/src/pages/Home.tsx
  - landing/scripts/capture/capture-scenario.md
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

"증거 필수"는 이 제품의 가장 핵심적인 약속을 한 장면으로 압축한 것이다 — 에이전트(또는 사람)가 작업을 "끝났다(done)"고 선언할 때, 그것을 뒷받침하는 증거를 함께 내지 않으면 시스템이 그 선언을 받아들이지 않는다. 즉 "done"은 스스로 외친다고 되는 것이 아니라, 근거가 있어야 성립한다.

랜딩 페이지의 히어로 데모는 이 한 순간을 위해 설계됐다. 데모 영상은 대시보드의 여섯 표면(요약 → 플랜 트리 → 백로그 → 타임라인 → 위키 → 보드)을 둘러본 뒤, 마지막에 보드에서 태스크를 "done"으로 옮기려는 장면에서 멈춘다. 증거 입력란이 비어 있는 동안 제출 버튼은 **비활성(disabled)** 상태로 막혀 있고, 도움말 힌트가 보인다. 사용자가 증거를 채우면 비로소 버튼이 활성화되고 태스크가 "done"으로 전환된다. 이 "비활성 제출 버튼 + 힌트" 프레임이 영상의 클라이맥스이자 포스터·OG 이미지가 겨냥하는 단 하나의 창이다.

이 약속은 두 층위에서 동일하게 성립한다:
- **화면(UX)**: 증거가 없으면 제출 버튼이 비활성으로 거부.
- **백엔드(계약)**: 같은 시도를 우회해도 거부(HTTP 400, EVIDENCE_REQUIRED) — 로컬 감사 기록에 남는다.

랜딩의 표어 자체가 이 약속을 그대로 옮긴다: "에이전트의 작업을 증거 없이 done으로 표시하길 거부하는 로컬 우선 데몬."

## 엔티티 (DB)

이 개념 자체는 별도 DB 엔티티가 아니다. 그 실체는 태스크의 done 전환 규칙에 있다 — 태스크가 종료 상태로 갈 때 증거 필드가 동반되어야 한다는 제약(불변식 `invariant.evidence-required-on-done`)이 데몬의 태스크 저장·전이 로직에 박혀 있다. 히어로 데모는 그 규칙을 사람이 눈으로 볼 수 있게 무대화한 것일 뿐, 새로운 데이터 구조를 만들지 않는다.

## API 표면

이 개념은 랜딩 페이지의 정적 표현(히어로 영상·포스터)으로만 노출된다. 영상 자체는 웹 대시보드의 보드 화면에서 상태 변경 모달을 조작해 촬영한 결과이며, 그 모달은 데몬의 태스크 done 전환 API를 호출한다. 랜딩 페이지에는 라이브 API가 없고, 미리 인코딩된 영상 파일(`/hero.mp4`, `/hero.webm`)과 포스터(`/hero-poster.png`)가 히어로 영역에 배선되어 있다.

## 불변식

- done(종료) 전환에는 증거가 필수다 — 증거 없는 done 시도는 화면에서 비활성 버튼으로 막히고 백엔드에서 거부된다(`invariant.evidence-required-on-done`).
- 히어로 영상의 클라이맥스 프레임은 "증거란이 빈 채 제출이 비활성"인 상태를 충분히 길게(약 1초 이상) 유지해야 보는 이가 그 의미를 인지한다 — 마케팅 자산의 무결성 조건.

## 구현 위치 (provenance)

- `landing/src/pages/Home.tsx`: 히어로 영역에 인코딩된 데모 영상(mp4/webm)과 포스터가 배선된 곳. 표어("증거 없이는 done을 거부", HTTP 400 EVIDENCE_REQUIRED)가 카피로 노출된다.
- `landing/scripts/capture/capture-scenario.md`: 이 한 순간을 촬영하기 위한 스토리보드·셀렉터 계약·기술 사양의 단일 진실 문서. 6개 표면 투어 후 보드에서 증거 필수 클라이맥스로 끝나도록 초 단위로 정의되어 있다.

증거 규칙 자체의 코드 실체는 데몬의 태스크 전이 로직에 있으며, 본 노드는 그 약속의 "히어로 무대화" 측면을 다룬다.

## 미확정 (OPEN)
- [ ] OPEN: 백엔드 거부의 정확한 에러 코드 문자열·응답 형은 데몬 측 코드/불변식 노드(evidence 계열)에서 확정 필요.
- [ ] OPEN: 히어로 영상이 가리키는 보드 상태 변경 모달의 셀렉터는 웹 하위 레포 변경에 따라 깨질 수 있어 촬영 전 재검증 대상.
