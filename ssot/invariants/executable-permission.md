---
id: invariant.executable-permission
kind: Invariant
title: 셸 스크립트 실행 권한 보존
definition: "install.sh·uninstall.sh·update-version-label.sh 세 셸 스크립트의 실행 권한이 편집·이동·리포맷·압축 해제 등 어떤 작업을 거쳐도 실행 가능 상태(mode 755)로 유지되어야 한다."
governs:
  - capability.copy-install-command
  - concept.hero-version-label
governs_note: ""
implementedIn:
  - landing/public/install.sh
  - landing/public/uninstall.sh
  - landing/scripts/update-version-label.sh
  - landing/.claude/rules/executable-permission-preserve.md
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:invariant"]
---

## 제약

랜딩 사이트가 가진 세 개의 셸 스크립트 — 사용자 설치 스크립트(install.sh), 제거 스크립트(uninstall.sh), 그리고 릴리스 때 hero 버전 라벨을 일괄 갱신하는 스크립트(update-version-label.sh) — 는 항상 "실행 가능" 상태로 남아 있어야 한다. 파일을 편집하거나, 옮기거나, 이름을 바꾸거나, 포매터를 돌리거나, 압축했다 풀더라도 실행 권한이 떨어지면 안 된다.

새 셸 스크립트를 추가할 때도 같은 커밋에서 실행 권한을 부여하고, 버전 관리 인덱스에까지 실행 가능으로 명시해야 한다.

## 깨지면 무슨 일이 일어나나

실행 권한이 떨어지면(읽기 전용 644 등으로 회귀하면) 그 스크립트를 직접 실행하는 경로가 깨진다. 가장 직접적인 피해는 버전 라벨 갱신 스크립트로, CI 나 릴리스 단계에서 실행을 시도하면 권한 거부로 실패하고 그 결과 hero 버전 라벨 자동 동기가 누락되어 버전 표기가 어긋난다. 설치 스크립트 자체는 공개 사이트에서 정적으로 서빙되므로 권한 손실이 사용자의 "한 줄 설치"를 즉시 깨지는 않지만, 저장소 안에서 그 스크립트를 직접 실행하는 로컬 검증·테스트 경로가 실패한다.

## 코드에서 어떻게 강제되나

세 스크립트는 현재 실행 가능 상태로 저장소에 들어 있고, 의도는 작성 규칙 문서(`landing/.claude/rules/executable-permission-preserve.md`)가 보존한다.

다만 강제 장치는 사실상 버전 관리 인덱스에 기록된 권한 비트 하나뿐이다. 권한 보존을 추가로 보장하는 속성 설정 파일이 없어, 다른 플랫폼에서 내려받거나(특히 압축 내보내기 후 풀기) 할 때 권한이 휘발될 수 있다. CI 는 권한을 검사하지 않고 빌드만 돌린다. 커밋 전 권한 회귀를 잡아주는 사전 검사 훅도 없다. 권한 변경은 변경 내역에서 한 줄로만 표시되어 사람 리뷰에서 놓치기 쉽다. 따라서 보존은 주로 작성자의 수동 확인 절차(커밋 전 실행 권한 확인)에 의존한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 실행 권한 보존을 명시적으로 정당화하는 전용 decision 노드가 인벤토리에 없다.
