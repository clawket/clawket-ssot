---
id: component.web
kind: SystemComponent
title: 웹 앱(@clawket/web)
definition: Clawket의 단일 페이지 웹 대시보드를 만드는 React 19 + Vite + Tailwind + dnd-kit 프런트엔드 패키지로, 빌드 결과물을 GitHub Release tarball로 배포해 데몬이 정적 서빙한다.
purpose: "사람 운영자가 LLM 에이전트와 동일한 작업 상태(프로젝트·플랜·유닛·사이클·태스크·지식)를 브라우저에서 보고 관리하도록, 데몬에 붙는 단일 페이지 대시보드를 빌드해 제공한다."
realizedBy:
  - capability.web-review
  - capability.wiki-knowledge
implementedIn:
  - web/package.json
  - web/index.html
  - web/vite.config.ts
  - web/src/main.tsx
dependsOn:
  - component.web-app-shell
  - component.web-api-client
  - component.daemon
consumesApi:
  - endpoint.daemon-events
  - endpoint.daemon-health
integratesWith:
  - integration.github-releases
impacts:
  - persona.human-operator
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
---

## 책임

웹 앱은 Clawket 운영자가 보는 화면 전체를 만들어 내는 프런트엔드 패키지다. 데몬이 들고 있는 작업 상태(프로젝트, 플랜, 유닛, 사이클, 태스크, 지식)를 브라우저에서 펼쳐 보여 주는 단일 페이지 애플리케이션이며, 사람 운영자가 LLM 에이전트와 같은 진실을 보고 손댈 수 있게 하는 표면이다.

이 패키지 자체는 "어떻게 화면을 조립하고, 어떻게 데몬과 통신하고, 어떻게 빌드·배포되는가"를 책임진다. 화면을 묶는 최상위 컨테이너 로직과 데몬 호출 로직은 각각 별도 하위 컴포넌트로 분리되어 있고, 이 노드는 그 둘을 포함해 한 덩어리의 배포 가능한 웹 산출물로 묶는 경계다.

기술 구성은 React 19를 코어로 하고, 번들러는 Vite, 스타일은 Tailwind, 칸반·백로그의 드래그앤드롭은 dnd-kit, 위키 본문 렌더링은 마크다운 계열 라이브러리를 쓴다. 라우팅은 해시 기반(HashRouter)이라 데몬이 정적 파일만 서빙해도 클라이언트 측 경로 전환이 깨지지 않는다.

## 경계와 의존

웹 앱은 자체 백엔드를 두지 않는다. 모든 데이터는 로컬 데몬(clawketd)이 단일 진실 저장소이며, 웹은 그 데몬의 HTTP/SSE 표면에만 의존한다. 따라서 이 컴포넌트는 데몬 컴포넌트에 의존하고, 화면 조립은 앱 셸 하위 컴포넌트, 데몬 호출은 API 클라이언트 하위 컴포넌트에 위임한다.

배포 경로는 일반적인 npm 패키지가 아니다. `web/dist/`를 GitHub Release tarball로만 배포하며, npm에서 끌어다 쓰는 것은 명시적으로 권장되지 않는다. 플러그인 설치 게이트가 이 tarball을 받아 풀어 두면 데몬이 그 정적 번들을 루트 경로로 서빙한다. 즉 운영 시점에는 웹과 데몬이 동일 출처(same-origin)에 놓인다.

개발(dev) 모드에서는 Vite 개발 서버(포트 5174)가 데몬의 여러 REST 경로를 프록시한다. 다만 SSE 스트림 경로만은 프록시 버퍼링 문제로 의도적으로 프록시에서 제외하고, 클라이언트가 데몬 출처로 직접 붙도록 빌드 타임 상수를 주입한다.

## 통신 패턴

데몬과의 통신은 두 갈래다. 첫째는 요청-응답형 REST로, 프로젝트·플랜·유닛·사이클·태스크·지식·코멘트·질문 등 거의 모든 상태 조작이 여기로 흐른다. 둘째는 서버-발신 이벤트(SSE) 스트림으로, 태스크·사이클·유닛·플랜·지식·코멘트의 변경을 실시간으로 받아 화면을 점진적으로 갱신한다. 데몬 생존 여부는 헬스 엔드포인트로 주기적으로 확인한다.

빌드된 번들은 GitHub Release tarball로 외부 배포 시스템과 맞물린다. 이 경로가 웹 앱이 외부와 닿는 유일한 배포 통합이다.

## 하위 서브패키지 (책임 단위)

- **앱 셸(component.web-app-shell)**: URL을 프로젝트/뷰/상세항목으로 해석하고, SSE 실시간 패치 리듀서·데몬 헬스·전역 단축키·상세 드로어·모달을 한데 묶는 최상위 컨테이너.
- **API 클라이언트(component.web-api-client)**: 데몬의 모든 REST 엔드포인트를 감싸는 fetch 기반 함수 모음. 쿠키·토큰 인증을 붙이고 에러를 정규화한다.

## 미확정 (OPEN)
- [ ] OPEN: 뷰별 컴포넌트(Board/Backlog/Timeline/Wiki/Summary)를 별도 SSOT 노드로 분리할지 여부는 인벤토리에 등록되지 않음.
