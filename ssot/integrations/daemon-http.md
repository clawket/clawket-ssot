---
id: integration.daemon-http
kind: Integration
title: 웹 대시보드 ↔ 데몬 HTTP API 연동
purpose: 웹 대시보드가 화면에 보여주고 바꾸는 모든 데이터를 로컬 데몬의 REST 엔드포인트로 읽고 쓰기 위한 단일 데이터 통로다. 별도 백엔드 서버 없이 브라우저가 곧장 로컬 데몬과 대화한다.
definition: 로컬에서 떠 있는 clawketd 데몬이 노출하는 HTTP REST 엔드포인트(프로젝트·플랜·유닛·사이클·태스크·지식·실행·질문·코멘트 등)를 브라우저의 fetch 로 호출하는 연동. 프로덕션은 데몬이 직접 웹 번들을 서빙해 같은 출처(same-origin)로 붙고, 개발 모드에서는 Vite 개발 서버의 프록시가 같은 경로를 데몬으로 중계한다.
integratesWith:
  - component.web-dashboard
  - component.daemon
implementedIn:
  - web/src/api.ts
  - web/src/lib/daemonUrl.ts
  - web/vite.config.ts
impacts:
  - concept.run
  - concept.knowledge
relatesTo:
  - to: integration.daemon-sse
    type: relates-to
    note: HTTP 는 요청-응답 읽기/쓰기, SSE 는 변경 푸시. 둘이 함께 화면을 최신으로 유지한다.
  - to: capability.web-review
    type: realizes
    note: 칸반 상태 변경·백로그 배정 등 대시보드의 모든 조작이 이 통로를 통해 데몬에 반영된다.
governedBy:
  - invariant.auth-cookie-token
  - invariant.daemon-serves-bundle
  - invariant.response-shape-backcompat
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드 옮겨적기 금지(식별자는 provenance/근거에만). -->

## 무엇과 연동하나

웹 대시보드(브라우저에서 도는 React 화면)는 자체 백엔드 서버를 두지 않는다. 사용자가 화면에서 보는 목록·상세·카운트, 그리고 칸반을 끌어 상태를 바꾸거나 백로그를 사이클에 배정하는 등의 변경은 모두 **로컬에 떠 있는 clawketd 데몬의 HTTP REST 엔드포인트**를 직접 호출해 처리한다. 프로젝트·플랜·유닛·사이클·태스크는 물론 지식(위키)·실행 기록·질문·코멘트·타임라인·실행 봉투까지, 대시보드가 다루는 거의 모든 데이터 종류가 이 한 통로를 거친다.

연결 방식은 실행 환경에 따라 두 갈래다.

- **프로덕션(배포)**: 데몬이 웹 번들 자체를 서빙하므로 브라우저 탭과 API 가 같은 출처에 놓인다. 따라서 API 호출은 도메인 없이 경로만으로 보내는 상대 경로 호출이면 충분하고, 데몬이 발급한 세션 쿠키가 같은 출처라 자연스럽게 함께 실린다.
- **개발 모드**: 별도 포트에서 도는 Vite 개발 서버가 데이터 경로(프로젝트·플랜·태스크·지식·실행 등)를 로컬 데몬으로 중계(프록시)한다. 데몬 주소는 명시적 환경변수가 있으면 그것을 쓰고, 없으면 캐시 디렉터리에 데몬이 적어둔 포트 파일을 읽어 찾으며, 그래도 없으면 기본 포트로 떨어진다.

인증은 데몬이 발급하는 세션 쿠키를 기본으로 하며, 개발 모드처럼 쿠키가 흐르지 않을 수 있는 경우를 대비한 헤더 기반 토큰이 보조 수단으로 함께 실린다.

## 구현 위치 (provenance)

- `web/src/api.ts` — 데몬 REST 와 대화하는 단일 클라이언트. 모든 엔티티별 호출 함수가 여기 모여 있고, 응답이 실패하면 데몬이 돌려준 본문에서 사람이 읽을 오류 메시지와 안정적 오류 코드를 뽑아내는 오류 래퍼를 둔다. 요청마다 쿠키를 포함시키고 보조 인증 헤더를 함께 붙인다.
- `web/src/lib/daemonUrl.ts` — 빌드 시점에 주입되는 데몬 출처 상수를 읽어, 프로덕션에서는 빈 문자열(상대 경로)로, 개발에서는 절대 출처로 URL 을 만든다.
- `web/vite.config.ts` — 개발 서버의 프록시 매핑 표. 데몬 주소를 환경변수→포트파일→기본 포트 순으로 해석하고, 데이터 경로들을 데몬으로 중계한다. (단, 실시간 이벤트 스트림 경로는 의도적으로 프록시 대상에서 제외된다 — 별도 연동 노드 참조.)

사용하는 내부 컴포넌트는 웹 대시보드(`component.web-dashboard`)이고, 상대편은 데몬(`component.daemon`)이다.

## 불변식

- 인증은 데몬이 발급한 세션 쿠키(보조로 토큰 헤더)에 의존한다 — `invariant.auth-cookie-token`.
- 프로덕션에서 데몬이 웹 번들을 직접 서빙해 같은 출처가 성립한다는 전제 — `invariant.daemon-serves-bundle`.
- 데몬이 응답 형태를 호환되게 유지해야 구버전/신버전 사이에서도 화면이 깨지지 않는다 — `invariant.response-shape-backcompat`. 실제로 일부 호출은 신설 엔드포인트가 없는 옛 데몬을 만나면 특정 상태 코드를 보고 빈 결과/대체 경로로 부드럽게 떨어진다.

## 영향 범위

이 연동이 닿는 데이터는 사실상 대시보드 전체다. 그중 실행 기록(`concept.run`)과 지식 항목(`concept.knowledge`)은 읽기뿐 아니라 화면에서의 가공·검색에도 직접 연결되므로 영향 추적의 출발점으로 명시한다. 이 통로가 막히거나 응답 형태가 바뀌면 대시보드의 거의 모든 뷰와 조작이 영향을 받는다.

## 미확정 (OPEN)
- [ ] OPEN: 보조 토큰 헤더가 개발 모드 외에도 쓰이는 경로가 있는지 확정 필요
- [ ] OPEN: 응답 호환성을 위해 폴백하는 엔드포인트의 전체 목록·상태 코드 정책 확정 필요
