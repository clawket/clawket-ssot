---
id: screen.tutorial
kind: Screen
title: 5분 튜토리얼 화면(정적 HTML)
purpose: 신규 사용자가 설치 직후부터 첫 태스크를 완료하기까지 약 5분 안에 Clawket의 구조화된 작업 루프(프로젝트→플랜 승인→유닛→사이클 활성화→태스크)를 직접 걸어보게 한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.landing-site
implementedIn:
  - landing/public/tutorial/index.html
  - landing/vercel.json
consumesApi: []
relatesTo:
  - to: endpoint.cli-init
    type: relates-to
    note: 프로젝트·플랜·유닛·사이클·첫 태스크 스캐폴딩을 한 번에 해주는 멱등 온보딩 명령과 같은 5분 첫 태스크 목표를 공유한다.
  - to: concept.project
    type: relates-to
  - to: concept.plan
    type: relates-to
  - to: concept.unit
    type: relates-to
  - to: concept.cycle
    type: relates-to
  - to: concept.task
    type: relates-to
  - to: endpoint.cli-replay
    type: relates-to
    note: 마지막 단계에서 완료한 태스크의 실행 이력을 리플레이로 되짚어보게 안내한다.
  - to: screen.spa-index
    type: relates-to
    note: SPA rewrite 규칙에서 /tutorial 경로가 제외되어 이 정적 HTML이 fallback 없이 직접 서빙된다.
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:screen
---

## 화면 목적
랜딩 사이트의 온보딩 튜토리얼 페이지("Your first task")다. 막 설치를 마친 사용자가 빈 상태에서 시작해 약 5분 만에 첫 태스크를 닫는 경험을 단계별로 안내한다. 목적은 기능 나열이 아니라 Clawket의 핵심 워크플로우 계층(프로젝트 → 플랜 승인 → 유닛 → 사이클 활성화 → 태스크 시작 → 완료)을 손으로 끝까지 한 번 통과시켜 "구조화된 에이전트 루프"를 몸에 익히게 하는 것이다.

## UI 요소 / 입력 필드
입력 요소가 없는 읽기 전용 가이드다. 자체 완결형 HTML(다크 테마, 단계별 소요 시간 배지)로 다음을 담는다.

- 도입부와 목차(Steps): 각 단계의 제목과 예상 소요 시간.
- 순차 단계 섹션(각 섹션에 단계 번호·소요 시간 표기):
  - 설치 + 데몬 확인 (30초)
  - 프로젝트 등록 (30초)
  - 플랜 생성 + 승인 (45초)
  - 유닛 생성 (15초)
  - 사이클 생성 + 활성화 (30초)
  - 첫 태스크 생성 (30초)
  - 태스크 시작 (10초)
  - 작업 수행 후 닫기 + 리플레이(가변)
- 각 단계의 실행 명령 예시(코드 블록)와 보충 설명 콜아웃.
- "다음으로 갈 곳" 마무리 섹션.

## 표시 데이터 / 호출 API
데몬 API를 호출하지 않는 정적 안내 문서다(consumesApi 없음). 전달하는 정보의 의미는 다음과 같다.

- 워크플로우 순서와 게이트: 플랜은 승인(active)되어야 태스크를 시작할 수 있고, 사이클은 활성화되어야 그 안의 태스크가 시작 가능하다는 순서 규칙을 실제 명령 흐름으로 보여준다.
- 각 엔티티의 역할: 프로젝트는 작업 디렉터리 바인딩, 플랜은 승인된 의도 컨테이너, 유닛은 그룹핑, 사이클은 타임박스 반복, 태스크는 직접 다루는 원자적 작업 단위.
- 닫기와 되짚기: 태스크를 완료한 뒤 실행 이력을 리플레이로 확인하는 마무리 흐름.

## 상태 / 엣지케이스
정적 HTML이라 로딩·빈·에러 상태나 권한 분기가 없다. 라우팅 측면: SPA fallback rewrite 규칙이 `/tutorial` 경로를 명시적으로 제외하므로 이 페이지는 SPA 인덱스로 흡수되지 않고 정적 파일 그대로 직접 서빙된다.

## 미확정 (OPEN)
- [ ] OPEN: 튜토리얼이 안내하는 명령 시퀀스가 CLI init 스캐폴딩과 정확히 어느 지점에서 겹치고 갈리는지(수동 단계 vs 자동 생성) 대조 확인할 것.
