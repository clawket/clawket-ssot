---
id: invariant.clawket-single-writer
kind: Invariant
title: 쓰기는 데몬 한 곳에서만 직렬화된다 (단일 writer)
definition: 저장소는 동시 읽기를 허용하되, 상태를 바꾸는 쓰기는 데몬 한 곳에서만 순서대로 처리된다. 여러 CLI 인스턴스나 동시에 터지는 서브에이전트 훅이 직접 쓰기를 시도하지 않고, 데몬이 단일 writer로서 모든 쓰기를 큐로 받아 직렬화한다.
governs:
  - component.daemon
  - component.sqlite-storage
  - component.cli
  - domain.work-management
implementedIn:
  - daemon/src/paths.rs
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 제약

상태 데이터를 바꾸는 쓰기 작업은 시스템 안에서 데몬이라는 단 하나의 주체를 통해서만, 그리고 한 번에 하나씩 순서대로 처리된다. 읽기는 여러 곳에서 동시에 해도 되지만, 변경은 반드시 데몬을 거쳐 줄 세워진다. CLI를 여러 개 띄우거나, Claude Code가 서브에이전트를 병렬로 여러 개 스폰해 훅이 동시에 발동하더라도, 그 모든 쓰기 요청은 데몬이 큐로 받아 차례대로 반영한다. CLI나 훅이 데이터베이스 파일에 직접 손을 대지 않는다는 것이 핵심이다.

## 깨지면 무슨 일이 일어나나

만약 여러 인스턴스가 저장소에 동시에 직접 쓰려 하면, 쓰기 락을 두고 충돌이 빈번해져 작업이 실패하거나 지연된다. 특히 서브에이전트가 병렬로 폭발적으로 늘어나는 상황에서는 훅이 한꺼번에 발동하므로 직렬화 지점이 없으면 데이터 경쟁과 부분 갱신, 상태 불일치가 발생할 수 있다. 단일 writer 보장이 깨지면 대시보드가 보여주는 상태의 일관성을 더 이상 신뢰할 수 없게 된다.

## 코드에서 어떻게 강제되나

데몬이 저장소를 단독으로 소유하고, 동시 읽기를 허용하는 저널 모드 위에서 쓰기를 한 지점으로 모아 직렬화한다. CLI와 어댑터 훅은 데몬에 요청을 보내는 클라이언트일 뿐, 저장소를 직접 열어 쓰지 않는다. 병렬 서브에이전트 훅의 동시 발동도 데몬이 큐로 수용해 순서대로 처리하므로, 외부에서 보면 변경이 항상 한 줄로 일어난 것처럼 보인다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 불변식을 도입한 결정 근거) 연결 필요
