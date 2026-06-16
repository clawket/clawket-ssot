---
id: invariant.daemon-serves-bundle
kind: Invariant
title: 프로덕션에서 웹 SPA 와 데몬 API/SSE 는 동일 origin 이다
definition: 웹 대시보드는 GitHub Release tarball 의 정적 빌드 산출물로 배포되어 데몬이 직접 정적 서빙하므로, 프로덕션에서 SPA·데이터 API·실시간 이벤트 스트림은 모두 같은 출처(origin)에서 제공된다.
governs:
  - component.web-dashboard
  - component.daemon
  - endpoint.daemon-events
implementedIn:
  - web/vite.config.ts
  - web/package.json
decidedBy:
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 자연어. 코드 옮겨적기 금지. -->

## 제약

웹 대시보드는 별도의 독립 웹서버에서 호스팅되는 것이 아니라, 빌드된 정적 파일 묶음(SPA 번들)을 GitHub Release 의 압축 산출물로 받아 로컬 데몬이 직접 내보내는 방식으로 동작한다. 그 결과 프로덕션 환경에서는 사용자가 보는 화면(SPA), 데이터를 읽고 쓰는 작업관리 API, 그리고 실시간 변경을 밀어 보내는 이벤트 스트림이 **모두 동일한 출처(같은 호스트·포트)에서 제공**된다. 즉 브라우저 탭이 떠 있는 주소와 데이터·이벤트를 가져오는 주소가 항상 같다.

이 동일 origin 전제 위에서 프런트엔드는 실시간 이벤트 구독을 절대 경로가 아니라 **상대 경로**로 건다. 빌드 모드에서는 데몬 주소가 빈 문자열로 주입되어 이벤트 경로가 상대 경로로 축약되며, 이는 "SPA 가 자신을 서빙해 준 바로 그 데몬에 다시 붙는다"는 동일 origin 전제를 코드 수준에서 굳힌 것이다.

## 깨지면 무슨 일이 일어나나

이 전제가 깨지면(예: 프로덕션 빌드에 데몬의 절대 주소를 주입하면) 실시간 이벤트 구독이 교차 출처 요청으로 바뀐다. 데몬은 자신을 `127.0.0.1` 로 알리는데 브라우저 탭은 `localhost` 에 있는 식으로 호스트 표기가 어긋나면, 한쪽에 묶인 세션 쿠키가 다른 쪽 요청에 실리지 않아 이벤트 스트림 연결 자체가 끊긴다. 결과적으로 보드·타임라인 등 화면이 변경을 실시간으로 반영하지 못하고 정지된 상태로 남는다.

개발 모드는 예외적으로 이 규칙을 우회한다. 개발용 프록시가 이벤트 스트림 응답을 버퍼링해 연결이 계속 "연결 중" 상태에 갇히는 문제가 있어, 개발 시에만 이벤트 구독을 데몬 주소로 직접 보내고(데몬의 교차 출처 허용 설정이 이를 받아준다) 나머지 API 만 프록시를 태운다. 따라서 "동일 origin" 은 프로덕션 배포 형상에 대한 불변식이고, 개발 모드의 우회는 의도된 예외다.

## 코드에서 어떻게 강제되나

빌드 설정(`web/vite.config.ts`)이 모드에 따라 데몬 주소 주입 값을 갈라 강제한다. 개발(serve)에서는 실제 데몬 주소를, 빌드에서는 빈 문자열을 주입해, 프로덕션 번들의 이벤트 경로가 상대 경로로 떨어지도록 한다. 배포 형상(정적 산출물을 데몬이 서빙) 자체는 패키지 메타데이터(`web/package.json`)가 "GitHub Release tarball 로 배포되는 React+Vite 대시보드"임을 명시하는 것으로 뒷받침된다. 데몬이 이 정적 번들을 서빙하는 라우팅 위치는 본 노드 범위에서 확정하지 못했다(OPEN).

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 동일 origin 결정의 근거 decision) 연결 필요
- [ ] OPEN: 데몬 측에서 정적 번들을 서빙하는 코드 위치 확인 필요(implementedIn 보강)
