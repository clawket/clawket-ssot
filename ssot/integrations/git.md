---
id: integration.git
kind: Integration
title: Git 작업트리 연동(엔벨로프 SHA·변경 파일 자동 채움)
definition: 데몬이 로컬에 설치된 git 명령을 얇게 감싸, 실행 봉투가 가리키는 대상 저장소의 현재 커밋(HEAD)과 변경 파일 목록을 읽어 봉투에 자동으로 채워 넣는 연동이다.
purpose: 에이전트가 실행 봉투를 만들 때 어떤 코드 상태에서 작업이 일어났는지를 사람 손을 거치지 않고 정확히 기록하기 위함 — 작업의 출발 커밋과 바뀐 파일을 자동으로 붙여 추적성을 높인다.
integratesWith:
  - component.daemon
implementedIn:
  - daemon/src/git.rs
impacts:
  - concept.execution-envelope
  - concept.task-envelope
relatesTo:
  - to: concept.execution-envelope
    type: relates-to
    note: 봉투의 계획 SHA·변경 파일 필드를 자동으로 채워 실행 계약을 보강한다
  - to: integration.git-destructive-guard
    type: relates-to
    note: 같은 git을 다루지만 이쪽은 읽기 전용 보강, 저쪽은 파괴적 명령 차단으로 역할이 다르다
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

연동 상대는 기기에 이미 설치된 git 명령 그 자체다. 별도 인증이나 원격 서버 통신은 없다 — 데몬은 등록된 프로젝트의 작업 디렉터리 안에서 git을 로컬로 실행해 두 가지 사실만 읽어 온다. 첫째는 현재 커밋 해시(HEAD)이고, 둘째는 두 커밋 사이에 바뀐 파일 이름 목록이다.

동작은 "논리적 이름 → 실제 폴더 찾기 → git 읽기" 순서다. 실행 봉투에는 대상 저장소를 가리키는 짧은 토큰(예: 폴더 이름)이 들어 있는데, 데몬은 등록된 프로젝트들의 작업 디렉터리 중에서 그 토큰과 폴더 이름이 정확히 일치하는 것을 찾는다(부분 일치가 아니라 정확 일치). 토큰에 `@HEAD` 같은 꼬리표가 붙어 있으면 떼어내고 본체로만 찾는다. 찾은 폴더에서 git으로 현재 커밋을 읽고, 필요하면 기준 커밋과 HEAD 사이의 변경 파일을 뽑는다.

이 연동은 실패에 관대하다. 폴더 이름이 안 맞거나, 그 폴더가 git 저장소가 아니거나, git이 깔려 있지 않거나, 명령이 실패하면 — 데몬은 멈추거나 오류를 던지지 않고 "값 없음"으로 처리한다. 호출하는 쪽은 이를 가벼운 경고로 받아들여, 계획 SHA를 비워 두고 경고 메모만 봉투에 덧붙인다.

## 구현 위치 (provenance)

git 호출 래퍼는 `daemon/src/git.rs`에 있다. 데몬(component.daemon)이 실행 봉투를 만들거나 채울 때 이 모듈을 불러 대상 저장소의 HEAD와 변경 파일을 가져온다. 본문만으로 재현하려면 이렇게 기억하면 된다 — "봉투 안의 저장소 토큰을 등록된 프로젝트 폴더 이름과 정확히 맞춰 찾고, 그 폴더에서 git으로 현재 커밋과 변경 파일을 읽되, 어떤 실패도 작업을 막지 않는다(부드러운 경고로 강등)."

같은 git을 다루는 또 다른 연동인 파괴적 명령 가드(integration.git-destructive-guard)와는 목적이 정반대다. 이쪽 연동은 git에서 사실을 읽어와 봉투를 풍부하게 만드는 읽기 전용 보강이고, 가드는 위험한 git 쓰기를 막는 차단 장치다.

## 불변식

이 연동 자체에 걸린 깨면 안 되는 제약은 현재 SSOT에 명시된 것이 없다. 다만 설계 원칙으로 "실패는 작업을 막지 않는다"가 일관되게 지켜진다 — git이 없거나 저장소가 아니어도 봉투 생성 자체는 계속 진행되고, 채우지 못한 값은 비워 둔 채 경고로 남긴다. 이는 강제 제약이라기보다 회복력 설계에 가깝다.

## 영향 범위

이 연동이 닿는 것은 실행 봉투다(concept.execution-envelope, concept.task-envelope). 자동으로 채워지는 계획 커밋·변경 파일은 봉투의 추적 정보가 되므로, 토큰 매칭 방식(정확 일치/꼬리표 처리)이나 채움 실패 정책이 바뀌면 봉투에 기록되는 출발 상태의 신뢰도가 함께 달라진다.

## 미확정 (OPEN)
- [ ] OPEN: 같은 폴더 이름을 가진 작업 디렉터리가 여러 프로젝트에 등록된 경우의 우선순위·모호성 해소 정책 확인 필요
