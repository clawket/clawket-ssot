---
id: endpoint.projects
kind: Endpoint
title: 웹 프로젝트 API (목록·조회·생성·수정·삭제·작업디렉터리)
definition: 웹 대시보드가 프로젝트를 조회·생성·수정·삭제하고 프로젝트에 묶인 작업 디렉터리(cwd)를 추가·제거하기 위해 데몬에 요청하는 API 표면이다.
realizedBy: [component.web-api-client]
implementedIn: [web/src/api.ts]
relatesTo:
  - { to: concept.project, type: mutates, note: 프로젝트의 생성·수정·삭제·작업디렉터리 편집을 수행한다 }
governedBy: []
impacts: [endpoint.daemon-projects]
consumedBy: [screen.web-project-settings, screen.web-summary, screen.sidebar]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
웹 대시보드의 화면들이 프로젝트를 다룰 때 호출하는 데이터 접근 묶음이다. 사람 운영자가 프로젝트 설정 화면이나 사이드바에서 프로젝트를 새로 만들거나 이름·설명·키·위키 경로를 고치거나, 어느 작업 디렉터리를 이 프로젝트에 속하게 할지 정할 때 이 경로를 통해 데몬과 대화한다. 데몬이 실제 저장소를 소유하므로 이 표면은 그 위에 얹힌 브라우저 측 호출 계층이다.

## 요청 / 응답
- 목록 조회: 등록된 모든 프로젝트를 돌려준다. 각 항목은 이름·설명·키·등록된 작업 디렉터리 목록·위키 경로·관리 활성화 여부 같은 프로젝트의 식별·설정 정보를 담는다.
- 단건 조회: 프로젝트 식별자 하나로 그 프로젝트의 같은 정보를 돌려준다.
- 생성: 최소한 프로젝트 이름을 보내며, 선택적으로 설명·키·첫 작업 디렉터리를 함께 보낸다. 만들어진 프로젝트를 돌려받고, 위키 경로나 추가 작업 디렉터리가 더 있으면 후속 수정 호출로 이어 붙인다.
- 수정: 바꿀 필드만 부분적으로 보낸다. 값을 비우려는 의도(지움)와 건드리지 않으려는 의도(생략)를 구분해 전달한다 — 이름·설명·키·작업 디렉터리·위키 경로·관리 활성화 토글을 바꿀 수 있다.
- 삭제: 프로젝트 식별자로 삭제하며 본문 없는 성공 응답을 받는다.
- 작업 디렉터리 추가/제거: 프로젝트에 디렉터리 경로 하나를 더하거나 빼며, 갱신된 프로젝트 상태를 돌려받는다.

## 권한 / 제약
호출은 데몬이 발급하는 세션 쿠키(브라우저) 또는 토큰 헤더(개발 서버 cross-port)로 인증된다. 데몬은 로컬 전용이므로 같은 기기의 사용자만 접근한다. 부분 수정에서 비우기와 생략을 구분하는 3-상태 표현을 쓰며, 관리 활성화 토글을 끄면 그 프로젝트에서 훅 강제가 무동작이 된다. 실제 검증·정합성 규칙은 데몬 측 프로젝트 엔드포인트가 책임진다.

## provenance
구현 위치는 web/src/api.ts 의 프로젝트 관련 호출(listProjects/getProject/createProject/updateProject/deleteProject/addProjectCwd/removeProjectCwd)이다. 이 표면을 제공하는 브라우저 측 컴포넌트는 component.web-api-client 이며, 실제 저장·검증은 데몬의 프로젝트 엔드포인트(endpoint.daemon-projects)가 수행한다. 코드가 사라져도 위 데이터 의미만으로 재구현 가능하도록, 필드 형태가 아니라 "무엇을 주고받는가"의 의미로 서술했다.

## 미확정 (OPEN)
- [ ] OPEN: 작업 디렉터리 제거가 마지막 하나일 때의 동작·검증, 키 중복 규칙 등 세부 정책은 데몬 측 구현에서 확인 필요.
