---
id: screen.web-redirect
kind: Screen
title: /web 리다이렉트 경로
definition: 데몬의 /web 주소로 접속하면 대시보드 진입 화면(루트 /)으로 영구 이동시키는 리다이렉트 경로다.
purpose: /web 같은 옛 주소나 습관적으로 입력한 경로로 들어온 사람 운영자를 실제 대시보드 진입점으로 흘려보낸다.
servesPersona: [persona.human-operator]
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/static_files.rs
consumesApi: []
relatesTo:
  - to: screen.spa-index
    type: leads-to
    note: /web 접속을 루트(/)의 대시보드 진입 화면으로 영구 이동시킨다.
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 화면 목적
사람 운영자가 데몬의 /web 주소로 접속했을 때, 별도의 화면을 그리지 않고 곧바로 대시보드 진입 화면(루트 /)으로 보내주는 경로다. 자체적으로 보여줄 내용이 없는 "통과 지점"이며, 옛 주소나 익숙한 경로로 들어온 사용자를 진짜 대시보드 입구로 안내하는 역할만 한다.

## UI 요소 / 입력 필드
보여주는 UI도 입력 필드도 없다. 접속 즉시 다른 주소로 넘기기 때문에 사용자가 머무를 화면이 존재하지 않는다.

## 표시 데이터 / 호출 API
표시하는 데이터가 없고 작업관리 API도 호출하지 않는다. 데몬은 이 경로 요청에 "영구 이동"에 해당하는 응답을 돌려주며, 이동 목적지로 루트(/) 주소를 지정한다. 브라우저는 이를 받아 자동으로 대시보드 진입 화면으로 다시 접속한다.

## 상태 / 엣지케이스
- 항상 동일 동작: 조건 분기 없이 언제 접속해도 루트(/)로 영구 이동시킨다.
- 영구 이동: "영구 이동"으로 응답하므로 브라우저나 중간 캐시가 이 경로를 루트로 가는 길로 기억할 수 있다. 따라서 이후 접속이 데몬을 거치지 않고 곧장 루트로 향할 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: /web 경로가 과거 어떤 주소 체계의 잔재인지(외부에 공유된 옛 링크 호환 목적인지) 배경 확인 필요.
