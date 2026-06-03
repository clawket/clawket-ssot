---
id: invariant.shell-script-content-type
kind: Invariant
title: 설치 스크립트 Content-Type 핀
definition: "공개 랜딩 사이트가 install.sh·uninstall.sh 를 내려줄 때 응답 헤더의 콘텐츠 종류가 항상 셸 스크립트(text/x-shellscript; charset=utf-8)로 고정되어, curl 한 줄 설치 경험이 브라우저·엣지의 임의 추측에 흔들리지 않아야 한다."
governs:
  - capability.copy-install-command
  - integration.vercel
  - integration.github-releases
governs_note: ""
implementedIn:
  - landing/vercel.json
  - landing/.claude/rules/shell-script-content-type-integrity.md
decidedBy:
  - decision.push-is-production
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:invariant"]
---

## 제약

방문자가 "한 줄 설치" 경로(`curl -fsSL .../install.sh | bash`)를 쓰면, 랜딩 사이트는 그 스크립트 파일을 내려줄 때 "이건 UTF-8 셸 스크립트"라는 종류 표시를 항상 똑같이 붙여야 한다. install.sh 와 uninstall.sh 두 파일 모두 동일하게 적용된다. 또한 이 콘텐츠 종류 고정과 짝을 이루는 사이트 전역 보안 기본값(브라우저가 종류를 멋대로 다시 추측하지 못하게 막는 nosniff, 레퍼러 정책, 권한 정책)도 함께 유지되어야 한다.

새로운 설치 스크립트 변종(예: 플랫폼별 install-mac.sh 같은 것)을 추가할 경우, 그 새 경로에도 동일한 종류 표시와 짧은 캐시 설정을 똑같이 등록해야 한다 — 한 종류만 표시가 붙고 다른 변종은 빠지는 비대칭이 생기면 안 된다.

## 깨지면 무슨 일이 일어나나

종류 표시가 빠지거나 다른 값(일반 텍스트·다운로드용 종류 등)으로 바뀌면, 호스팅 엣지가 `.sh` 파일을 자기 나름대로 추측해서 내려준다. 그 결과 일부 환경에서는 설치 명령이 스크립트를 실행하지 못하고 그냥 파일 다운로드로 처리되거나, 인코딩·권한 문제로 설치가 깨진다. 즉 제품의 첫 접점인 "한 줄 설치" 경험이 사용자 환경에 따라 들쭉날쭉하게 실패한다. 짝을 이루는 보안 기본값까지 함께 제거되면, 종류 고정이 의도대로 동작하지 않을 수 있다(브라우저가 다시 추측을 시작).

## 코드에서 어떻게 강제되나

이 제약은 랜딩 사이트의 Vercel 응답 헤더 정의(`landing/vercel.json`)에 박혀 있다. install.sh·uninstall.sh 각각에 셸 스크립트 종류 표시와 짧은 캐시(5분)가 명시되고, 사이트 전역 항목에 보안 기본값이 함께 정의된다. 의도와 그 배경은 별도의 작성 규칙 문서(`landing/.claude/rules/shell-script-content-type-integrity.md`)가 보존한다.

다만 자동 강제 장치는 없다. CI 는 빌드만 돌리고 이 헤더 정의의 내용·정합성을 검사하지 않는다. 잘못된 설정 문법을 호스팅이 조용히 무시할 수 있어, 파일이 망가져도 빌드는 통과할 수 있다. 따라서 변경 후 production 응답을 실제로 확인하는 검증(배포 뒤 헤더를 직접 조회해 종류 표시가 셸 스크립트인지 보는 절차)은 사람이 수동으로 수행하는 절차이며, 자동 배포 게이트는 없다.

## 미확정 (OPEN)
- [ ] OPEN: 이 콘텐츠 종류 고정을 명시적으로 정당화하는 전용 decision 노드가 인벤토리에 없다(현재는 배포 게이트 결정에 임시 연결). 전용 결정 기록 필요 시 연결.
