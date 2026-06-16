---
id: concept.envelope
kind: Concept
title: 봉투(웹 폼 관점)
definition: "웹 대시보드가 태스크 상세에서 편집·검증하는 ADR-0001 실행 봉투의 화면 표상으로, 19개 정규 필드를 정해진 순서·위젯으로 렌더링하고 데몬 검증과 상속 해석을 거쳐 활성 봉투로 서명한다."
relatesTo:
  - to: concept.execution-envelope
    type: relates-to
    note: 데몬·CLI가 정의하는 19필드 실행 봉투를 웹 폼이 그대로 투영한다 — 봉투의 진실은 데몬에 있고 이 노드는 그 편집/표시 표면이다.
  - to: concept.task
    type: belongs-to
    note: 봉투는 항상 한 태스크에 매달려 있으며 태스크 상세 화면 안에서만 편집된다.
  - to: concept.envelope-version-history
    type: relates-to
    note: 봉투를 서명할 때마다 새 버전이 쌓여 버전 히스토리를 이룬다.
  - to: endpoint.daemon-search
    type: relates-to
    note: 봉투 자체는 검색 대상이 아니나 같은 데몬 HTTP 표면을 통해 읽고 서명된다.
governedBy: []
governs: []
dependsOn:
  - concept.execution-envelope
implementedIn:
  - web/src/types.ts
  - web/src/components/EnvelopeForm.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 자연어. 코드 옮겨적기 금지. 분기/의도 불명은 OPEN. -->

## 정의

봉투(웹 폼 관점)는 사람 운영자가 웹 대시보드의 태스크 상세 화면에서 보고 고치는 **실행 봉투의 화면 표상**이다. 실행 봉투 자체의 의미와 권위는 데몬·CLI가 정의하는 19필드 계약(`concept.execution-envelope`)에 있으며, 이 노드는 그것을 사람이 다루기 위한 **편집·검증 UI**를 가리킨다.

화면은 ADR-0001 이 규정한 **19개 정규 필드를 고정된 렌더 순서**로 보여준다: 봉투 버전, 의도(intent), 대상 저장소, 대상 모델, 최대 턴 수, 프롬프트 템플릿, 컨텍스트 참조, 범위 경계(scope_boundary), 원자 크기 힌트, 성공 기준(success_criteria), 검증 명령, 의존, 차단, 계획된 커밋(planned_sha), 분해 정책(decomposition_policy), 체크포인트 간격, 롤백 전략, 출처(origin), 배정 모델이다. 폼은 **이 19개만** 입력으로 노출하며, 봉투 JSON에 다른 키가 들어와도 화면에는 표시하지 않고 저장 시 그대로 통과시킨다(라운드트립).

각 필드는 의미에 맞는 입력 위젯으로 그려진다 — 한 줄 텍스트, 여러 줄 텍스트, 숫자, 줄 단위 목록(예: 컨텍스트 참조·범위 경계·성공 기준·의존), 그리고 정해진 보기에서 고르는 선택(원자 크기 힌트는 tiny/small/medium/large, 분해 정책은 auto/manual/atomic)이다. 계획된 커밋(planned_sha)은 읽기 전용으로만 표시된다.

편집 중에는 입력이 멈춘 뒤 짧은 디바운스(약 0.4초) 후 데몬에 **초안 검증**을 요청해, 필드별 오류·경고(에러/경고 두 등급)를 실시간에 가깝게 보여준다. 저장(서명)하면 데몬이 봉투의 새 버전을 기록하고, 그 결과로 활성 봉투가 갱신된다.

## 엔티티 (DB)

봉투의 저장은 데몬 측 책임이다. 웹 관점에서 다루는 데이터 모양은: 봉투 응답이 원본 봉투(raw), 상속이 해석된 봉투(resolved), 상속 체인, 버전 번호, 폐기 여부(superseded)를 함께 싣는다. 상속 체인이 비어 있지 않으면 부모 태스크의 봉투 값이 자식에 흘러내려 합쳐진 결과가 resolved 로 표시된다. 봉투 값은 필드마다 문자열·숫자·배열·객체 등 임의의 JSON 모양을 가질 수 있고, 폼은 제출 시 필드별로 형 변환(문자열→숫자, 줄바꿈 텍스트→배열 등)을 수행한다.

## API 표면

- 활성 봉투 조회: 태스크의 현재 봉투를 가져온다(봉투가 아직 없으면 없음으로 처리, 오류로 보지 않음). `resolve` 옵션으로 상속 해석본을 받을 수 있다.
- 봉투 서명: 태스크에 새 봉투 버전을 기록한다. 여기에 기록된 봉투가 새 활성 봉투가 된다.
- 봉투 삭제: 태스크의 봉투를 제거한다.
- 초안 검증: 저장하지 않은 초안 봉투를 데몬 구조 규칙에 대조해 위반 목록을 받는다. 봉투가 비어 있고 태스크에 활성 봉투도 없으면 검증 대상 없음으로 처리한다.

## 불변식

- 봉투는 **19개 정규 필드만** 입력 표면으로 노출한다. 추가 키는 보존하되 화면에 끌어올리지 않는다.
- 활성 봉투는 항상 가장 최근에 서명된 버전이다 — 서명이 곧 활성 봉투 교체다.
- 봉투는 단독으로 존재할 수 없으며 반드시 한 태스크에 종속된다.

## 구현 위치 (provenance)

- `web/src/types.ts` — 19개 정규 필드 목록(ENVELOPE_FIELDS, 렌더 순서)과 원자 크기 힌트·분해 정책 보기값, 봉투 응답/검증 결과의 데이터 모양 선언.
- `web/src/components/EnvelopeForm.tsx` — 19필드를 위젯으로 그리는 폼, 필드별 도움말, 디바운스 후 데몬 초안 검증 호출, 제출 시 필드별 형 변환.

## 미확정 (OPEN)
- [ ] OPEN: 19필드 각각의 데몬 측 검증 규칙(필수/형식)의 구체 내용은 데몬 소스 기준으로 별도 확인 필요.
