---
id: component.renderer
kind: SystemComponent
title: 데스크탑 렌더러(React 프론트엔드)
definition: 데스크탑 앱의 화면을 그리는 React + Vite 프론트엔드. 사이드바·상단바·상세 드로어·명령 팔레트로 짜인 셸 위에 여러 뷰를 띄우고, 데몬과 HTTP 및 실시간 이벤트로 통신해 작업 상태를 보여 준다.
purpose: Tauri 웹뷰 안에서 작업관리 화면(요약·보드·백로그·타임라인·위키 등)을 렌더링하고, 데몬과 통신하며 명령 팔레트·전역 단축키·데몬 연결 상태 표시를 제공한다.
realizedBy:
  - capability.web-review
servesPersona:
  - persona.human-operator
implementedIn:
  - desktop/apps/desktop/src/App.tsx
  - desktop/apps/desktop/src/main.tsx
  - desktop/apps/desktop/package.json
dependsOn:
  - component.data-provider
  - component.ui-package
  - component.tauri-shell
consumesApi:
  - endpoint.daemon-health
providesApi: []
integratesWith: []
impacts:
  - component.desktop-app
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
---

## 책임

렌더러는 Tauri 웹뷰 안에서 도는 React 프론트엔드로, 데스크탑 앱이 사람에게 보여 주는 모든 화면을 그린다. 화면 골격은 사이드바·상단바·본문·상세 드로어로 이뤄진 셸이며, 본문에는 여러 뷰(요약·보드·백로그·타임라인 등) 중 하나가 활성 뷰로 떠 있다. 상단바에서 뷰를 바꾸거나, 사이드바의 플랜 트리에서 항목을 골라 상세를 열 수 있다.

사이드바에는 플랜·유닛·태스크가 트리 형태로 모여 있어 작업 구조를 한눈에 보여 준다. 트리의 한 노드를 고르면 그 항목이 선택 상태가 되고, 선택은 프로젝트를 바꾸면 자동으로 초기화된다.

명령 팔레트는 단축키로 열리는 빠른 이동 수단이다. 뷰·플랜·유닛·태스크를 한 목록에서 검색해 골라 이동할 수 있고, 검색어가 없을 때는 목록이 너무 길어지지 않도록 무거운 그룹의 개수를 적당히 잘라 보여 준다. 태스크는 사람이 읽는 티켓 번호로도 찾을 수 있다. 사이클도 목록에 올라오긴 하지만, 현재 선택 모델에 사이클 항목이 아직 없어 사이클로의 직접 이동은 후속 작업으로 미뤄져 있다.

또한 렌더러는 데몬과의 연결이 살아있는지를 계속 살펴, 끊기면 상단의 연결 표시를 경고로 바꾸고 재연결을 시도하며 사용자에게 알림(토스트)을 띄운다. 재연결 시에는 데이터 새로고침도 함께 건다. 도움말 모달과 전역 단축키도 렌더러가 관리한다.

## 경계와 의존

렌더러는 데스크탑 앱(`component.desktop-app`)의 화면 계층이며, 사람 운영자(`persona.human-operator`)가 마주하는 표면이다. 같은 앱 안에서 네이티브 셸(`component.tauri-shell`)이 데몬 기동과 토큰 제공을 맡고, 렌더러는 그 위에서 화면만 책임진다.

데이터는 직접 데몬을 호출하지 않고 데이터 공급자(`component.data-provider`)를 통해 한 벌의 진실로 받아 쓴다. 화면을 구성하는 버튼·뱃지·앱 셸·플랜 트리·태스크 카드·명령 팔레트 같은 부품은 공용 UI 패키지(`component.ui-package`)에서 가져온다. 데몬 연결 상태 확인은 데몬 헬스(`endpoint.daemon-health`)에 기대며, 이 확인 자체도 공급자가 노출하는 헬스 함수를 통해 이뤄진다.

## 통신 패턴

렌더러는 데몬과 직접 마주하기보다 데이터 공급자를 거쳐 통신한다. 공급자가 데몬 HTTP API로 읽기·쓰기를 라우팅하고 실시간 이벤트를 구독하므로, 렌더러는 그 결과 상태를 받아 그리고 사용자 동작을 공급자의 함수 호출로 흘려보낸다. 연결 생존 표시는 주기적인 헬스 확인 결과를 상단바 표시와 토스트로 반영한다.

## 하위 서브패키지 (책임 단위)

- 앱 진입·셸 조립: 사이드바·상단바·본문·상세 드로어를 엮고 활성 뷰를 전환하는 부분.
- 명령 팔레트: 뷰·플랜·유닛·태스크를 검색해 이동하는 빠른 탐색 표면.
- 전역 단축키·도움말: 키 입력으로 팔레트·도움말을 여는 부분.
- 데몬 연결 상태 훅: 헬스 확인으로 연결 생존을 감지해 표시·알림을 거는 부분.
- 마운트 진입점: 테마를 초기화하고 앱을 화면에 그리는 부트스트랩.

## 미확정 (OPEN)
- [ ] OPEN: 첫 실행 시 활성 프로젝트가 없을 때의 폴백 프로젝트 식별자가 실제 환경에서 어떻게 해소되는지 확인 필요
- [ ] OPEN: 사이클 선택·이동 지원이 후속에 추가되었는지 확인 필요
