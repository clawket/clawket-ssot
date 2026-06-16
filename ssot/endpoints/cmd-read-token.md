---
id: endpoint.cmd-read-token
kind: Endpoint
title: 데스크탑 read_token 명령 (세션 토큰 읽기)
definition: "데스크탑 화면(렌더러)이 데몬에 접속할 때 필요한 인증 토큰을 디스크에서 읽어 돌려주는 내부 명령. 토큰 파일 내용을 문자열로 반환하거나, 없거나 비어 있으면 사람이 읽을 수 있는 에러 문구를 돌려준다."
realizedBy:
  - component.tauri-shell
implementedIn:
  - desktop/apps/desktop/src-tauri/src/token.rs
relatesTo:
  - to: integration.session-token
    type: reads
    note: "데몬이 기동 시 캐시 디렉토리에 써 둔 회전 세션 토큰 파일을 읽는다."
  - to: component.renderer
    type: backed-by
    note: "렌더러가 데몬 호출 직전 이 명령으로 토큰을 가져와 요청 헤더에 실어 보낸다."
governedBy:
  - invariant.auth-cookie-token
impacts:
  - component.renderer
consumedBy:
  - component.renderer
  - component.daemon-client
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

데스크탑 앱의 화면(웹뷰 기반 렌더러)은 보안 격리 때문에 사용자 캐시 폴더에 직접 접근할 수 없다. 그런데 데몬은 자기 자신을 보호하기 위해 매 요청마다 세션 토큰을 요구한다(로컬 TCP 접속 보호). 이 간극을 메우는 것이 `read_token` 명령이다.

렌더러가 데몬에 첫 요청을 보내기 직전, 데스크탑 셸(네이티브 측)에 "지금 유효한 세션 토큰이 뭐냐"고 물어본다. 셸은 데몬이 기동할 때 디스크에 써 둔 토큰 파일을 읽어 그 내용을 돌려준다. 렌더러는 받은 토큰을 한 번 기억해 두었다가, 이후 모든 데몬 요청의 인증 헤더로 재사용한다. 인증이 거절(401)되면 기억한 토큰을 버리고 다음 요청 때 다시 이 명령으로 새로 읽어 온다.

이 명령은 사용자가 직접 부르는 것이 아니라, 데스크탑 앱 내부에서 화면과 네이티브 셸 사이를 잇는 통로로만 쓰인다.

## 요청 / 응답

- 요청: 입력값이 없다. 렌더러가 "토큰을 달라"고 부르기만 하면 된다.
- 정상 응답: 토큰 문자열. 파일 끝의 공백·줄바꿈은 잘라낸 깔끔한 값으로 돌려준다.
- 실패 응답: 사람이 읽을 수 있는 에러 문구. 두 경우로 나뉜다 — (1) 토큰 파일이 비어 있을 때("토큰 파일이 비었음" 취지), (2) 파일을 읽지 못했을 때(아직 데몬이 안 떴거나 경로가 없는 경우, 어느 경로에서 실패했는지 포함). 이 에러는 렌더러로 그대로 전달되어, 화면은 "데몬에 연결할 수 없음" 류의 상태로 안내할 근거가 된다.

토큰 파일의 위치는 환경 설정을 우선 따르고(전용 캐시 경로 지정이 있으면 그것, 없으면 표준 캐시 폴더, 그것도 없으면 홈 밑 기본 캐시 폴더), 데몬이 토큰을 쓰는 위치와 동일한 규칙으로 찾는다. 즉 "데몬이 쓰는 곳"과 "셸이 읽는 곳"이 같은 약속을 공유한다.

## 권한 / 제약

- 데스크탑 앱 내부 전용 통로다. 외부에서 호출할 수 있는 공개 API가 아니다.
- 토큰이 존재하고 비어 있지 않아야 정상으로 본다. 빈 토큰이나 읽기 실패는 모두 에러로 취급해, 인증 없이 데몬에 접근하는 일이 생기지 않게 막는다.
- 토큰은 회전(주기적으로 갱신)되므로, 한 번 읽은 값이 영구히 유효하지 않다. 인증 거절을 만나면 다시 읽는 흐름이 전제된다.
- 토큰 인증의 의미·강제 규칙 자체는 별도 불변식이 정의한다.

## provenance

- 명령 구현: `desktop/apps/desktop/src-tauri/src/token.rs` (네이티브 셸 측 `read_token`).
- 호출·재사용 흐름: `desktop/apps/desktop/src/data/api.ts` (렌더러의 토큰 로더가 이 명령을 호출하고, 받은 값을 캐시해 매 요청 헤더로 실음).
- 제공 컴포넌트: `component.tauri-shell` (네이티브 셸).
- 토큰의 실제 파일 위치·쓰기 주체·생애는 `integration.session-token` 참조.

## 미확정 (OPEN)
- [ ] OPEN: 토큰 회전 주기와 갱신 시점(데몬 재기동 시마다인지, 시간 기반인지)을 데몬 측 코드에서 확인.
- [ ] OPEN: 빈 토큰 / 읽기 실패 각각에 대해 화면이 보여 주는 안내 문구가 다른지(상태 구분) 확인.
