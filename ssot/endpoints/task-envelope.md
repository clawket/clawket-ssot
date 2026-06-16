---
id: endpoint.task-envelope
kind: Endpoint
title: 실행 봉투 조회 · 서명 · 삭제 · 검증 · 이력
definition: 태스크의 활성 실행 봉투를 조회·서명(갱신)·삭제하고, 초안 봉투를 검증하며, 봉투 버전 이력을 보는 웹 대시보드용 봉투 관리 묶음.
realizedBy: [component.daemon]
implementedIn:
  - web/src/api.ts
  - web/src/components/EnvelopeForm.tsx
relatesTo:
  - { to: concept.execution-envelope, type: mutates, note: "활성 봉투를 읽고, 새 버전으로 서명하고, 비우고, 초안을 검증한다" }
  - { to: concept.task, type: reads, note: "봉투는 특정 태스크에 부속된다" }
governedBy: []
impacts: []
consumedBy: [component.web-dashboard, component.web-api-client, screen.task-detail-drawer]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

태스크에 부속된 실행 봉투(그 태스크를 어떤 의도·성공 기준·분해 정책으로 수행할지 적은 실행 계약)를 관리하는 기능 묶음이다. 조회(현재 활성 봉투 보기), 서명(새 봉투 버전 쓰기), 삭제(봉투 비우기), 검증(아직 저장하지 않은 초안 봉투가 형식 규칙에 맞는지 미리 점검), 이력(봉투가 바뀌어 온 버전 기록 보기)을 포함한다. 웹 대시보드의 봉투 편집 폼에서, 사람이 태스크의 실행 계약을 작성·수정·점검할 때 쓰인다.

## 요청 / 응답

조회는 대상 태스크를 지정하고, 봉투를 그대로 볼지 아니면 상속까지 풀어낸(resolve) 형태로 볼지, 특정 버전을 볼지 등을 선택할 수 있다. 아직 봉투가 없는 태스크를 조회하면 오류가 아니라 "없음"으로 다뤄, 화면이 빈 폼을 그릴 수 있게 한다.

서명은 새 봉투 내용을 보내면 그 태스크에 새 봉투 버전을 기록하고, 그 버전이 새 활성 봉투가 된다. 삭제는 태스크의 봉투를 비운다.

검증은 아직 저장하지 않은 초안 봉투를 보내 데몬의 구조 규칙에 맞는지 미리 확인한다(strict 여부, 상속 풀이 여부를 곁들일 수 있다). 봉투를 생략했고 그 태스크에 활성 봉투도 없으면 "검증할 대상 없음"으로 다뤄야 하며 오류로 취급하지 않는다.

이력은 그 태스크 봉투의 버전 기록을 최신순으로 돌려준다. 이 중 아직 다음 버전으로 교체되지 않은(superseded 되지 않은) 단 하나가 현재 활성 봉투다. 이 이력은 타임라인의 리플레이 화면에서 변경 흐름을 재생하는 데 쓰인다.

## 권한 / 제약

- 조회·검증·이력은 읽기 성격이고, 서명·삭제는 봉투 상태를 바꾸는 쓰기다.
- "활성 봉투는 하나"라는 규칙이 핵심이다 — 새 서명은 이전 활성 봉투를 다음 버전으로 밀어내고 자신이 활성이 된다. 이력에서 활성 봉투는 교체되지 않은 단 하나로 식별된다.
- 봉투 없음 / 검증 대상 없음을 오류가 아닌 정상 "빈 상태"로 다루는 것이 약속된 동작이다.
- 봉투 구조 규칙(검증 기준)의 단일 진실 주체는 데몬이다.

## provenance

웹 클라이언트(`web/src/api.ts`)가 데몬의 봉투 경로(`/tasks/{id}/envelope`, `/envelope/validate`, `/envelope/history`)를 감싸고, `web/src/components/EnvelopeForm.tsx`가 봉투 편집·검증 UI를 제공한다. 봉투 버전 관리·검증 규칙·활성 봉투 식별은 데몬(component.daemon)이 책임진다. 코드가 사라져도 "태스크 봉투의 조회·서명·삭제·초안 검증·버전 이력"이라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: strict 검증과 상속 풀이(resolve)가 검증 결과에 구체적으로 어떤 차이를 만드는지 데몬 봉투 모듈에서 확인해 보강할 것.
