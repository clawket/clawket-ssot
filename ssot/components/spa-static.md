---
id: component.spa-static
kind: SystemComponent
title: SPA 정적 서빙(웹 대시보드 호스팅)
definition: 데몬이 빌드된 React 대시보드 번들을 같은 프로세스에서 직접 내려주는 정적 파일 서빙 단위로, 번들이 빌드되어 있으면 SPA 진입 화면과 자산을 응답하고 빌드되어 있지 않으면 빌드 방법을 안내하는 대체 화면을 돌려준다.
purpose: 데몬과 별도의 웹 서버 없이도 사람 운영자가 브라우저로 작업 상태를 볼 수 있도록, 빌드된 웹 대시보드와 그 자산을 데몬이 직접 서빙한다.
realizedBy:
  - capability.web-review
implementedIn:
  - daemon/src/routes/static_files.rs
dependsOn:
  - component.clawketd
  - component.http-router
consumesApi: []
providesApi:
  - endpoint.daemon-health
integratesWith: []
impacts:
  - component.web-dashboard
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
relatesTo:
  - to: component.web-dashboard
    type: relates-to
    note: 빌드된 React SPA 번들(진입 화면·자산)을 브라우저에 내려주는 호스트 역할.
  - to: invariant.daemon-serves-bundle
    type: relates-to
    note: 데몬이 웹 번들을 직접 서빙한다는 불변식을 구현하는 실행 지점.
  - to: endpoint.cli-web-views
    type: relates-to
    note: timeline/board/wiki/summary 등 웹 뷰 단축 명령이 여는 화면의 실제 서빙 경로.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 책임

이 단위는 데몬이 "작업 관리용 API 서버"인 동시에 "웹 대시보드를 직접 내려주는 웹 호스트"가 되도록 만드는 부분이다. 별도의 정적 웹 서버나 외부 호스팅을 두지 않고, 사람 운영자가 브라우저 주소창에 데몬 주소만 입력하면 곧바로 대시보드 화면이 뜨도록 하는 것이 핵심 책임이다.

구체적으로 맡는 일은 네 가지다. 첫째, 최상위 진입 요청에 대해 대시보드의 시작 화면(SPA 진입 페이지)을 돌려준다. 둘째, 과거 호환을 위한 `/web` 같은 경로를 최상위로 영구 이동(redirect)시켜 주소를 하나로 모은다. 셋째, 파비콘·아이콘 같은 브랜드 자산과 SPA가 필요로 하는 묶음 자산(스크립트·스타일 등)을 내려준다. 넷째, 대시보드가 아직 빌드되지 않은 환경에서는 깨진 화면 대신 "대시보드가 빌드되지 않았다"는 안내와 빌드 명령을 담은 대체 페이지를 돌려준다.

## 경계와 의존

이 단위는 데몬(`component.clawketd`) 프로세스 안에서 데몬의 HTTP 라우터(`component.http-router`)에 묶여 동작한다. 즉 독립 실행 컴포넌트가 아니라 데몬 HTTP 표면의 한 갈래다. 내려줄 파일의 실제 위치는 데몬이 알고 있는 웹 번들 디렉터리(설치된 `web/dist`에 해당)이며, 그 디렉터리가 설정되어 있지 않거나 진입 파일이 없으면 곧바로 대체 안내 화면으로 전환한다.

자산 요청을 처리할 때는 요청 경로가 자산 루트 디렉터리 바깥을 가리키지 못하도록 경로를 정규화해 루트 안쪽인지 확인한 뒤에만 파일을 읽는다. 이는 임의 파일 노출(경로 탈출)을 막기 위한 경계 검사다. 또한 진입 화면 응답에는 브라우저가 데몬 인증 채널을 자동으로 갖추도록 세션 토큰을 담은 쿠키(자바스크립트가 읽지 못하는 HttpOnly, 동일 사이트 한정)를 함께 내려, 이후 대시보드가 보내는 요청이 별도 설정 없이 인증되도록 한다.

## 통신 패턴

브라우저와 일반 HTTP로 통신한다. 진입 화면과 자산은 정적 파일 응답이며, 자산처럼 변하지 않는 파일에는 장기 캐시를 허용하되, 진입 화면 자체는 재시작마다 갱신되는 세션 토큰 쿠키 때문에 캐시하지 않도록(no-store) 응답해 낡은 토큰이 브라우저에 굳는 일을 막는다. 대시보드가 빌드되어 있지 않을 때의 안내 응답은 정상 화면이 아님을 알리는 "서비스 일시 불가" 형태로 내려준다.

작업 데이터 자체는 이 단위가 다루지 않는다. 화면을 받은 브라우저는 이후 데몬의 작업 관리 API와 실시간 이벤트 스트림에 직접 붙어 데이터를 주고받는다.

## 하위 서브패키지 (책임 단위)

별도의 하위 패키지로 나뉘지 않는 단일 라우팅 모듈이다. 내부적으로는 진입 화면 처리, 자산 처리(경로 경계 검사 포함), 미빌드 대체 화면 생성, 파일별 콘텐츠 유형 판별 정도의 역할 함수로 구성된다.

## 미확정 (OPEN)
- [ ] OPEN: 진입 화면 응답에 실리는 세션 토큰 쿠키와 데몬 인증 계층(토큰 헤더 수용)의 전체 연동 범위는 본 단위 밖이라 별도 확인 필요.
