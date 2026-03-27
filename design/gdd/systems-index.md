# Systems Index: 블록 원정대 (Block Expedition)

> **Status**: Draft
> **Created**: 2026-03-26
> **Last Updated**: 2026-03-26
> **Source Concept**: design/gdd/game-concept.md

---

## Overview

블록 원정대는 테트리스 코어 위에 캠페인 구조와 해금 시스템을 얹은 2D 퍼즐 게임이다.
그리드 기반 보드와 입력 처리가 기반이 되고, 그 위에 미션 조건 시스템이 매 스테이지의
다양성을 만든다. 스킬 시스템과 캠페인 해금이 장기 성장감을 제공하며, 각 게임플레이
시스템에 대응하는 UI 레이어가 정보를 플레이어에게 전달한다.
총 14개 시스템, 2주 개발 일정 기준으로 MVP 7개 → Vertical Slice 3개 → Alpha 3개 →
Full Vision 1개 순으로 설계·구현한다.

---

## Systems Enumeration

| # | System Name | Category | Priority | Status | Design Doc | Depends On |
|---|-------------|----------|----------|--------|------------|------------|
| 1 | 게임 보드 | Core | MVP | Designed | design/gdd/game-board.md | — |
| 2 | 입력 시스템 | Core | MVP | Approved | design/gdd/input-system.md | — |
| 3 | 씬 관리자 | Core | MVP | Designed | design/gdd/scene-manager.md | — |
| 4 | 테트리스 코어 | Gameplay | MVP | Designed | design/gdd/tetris-core.md | 입력 시스템, 게임 보드 |
| 5 | 미션 조건 시스템 | Gameplay | MVP | Designed | design/gdd/mission-condition-system.md | 테트리스 코어, 게임 보드 |
| 6 | 게임 HUD | UI | MVP | Not Started | — | 테트리스 코어, 미션 조건 시스템, 스킬 시스템 |
| 7 | 게임 오버 화면 | UI | MVP | Not Started | — | 씬 관리자, 테트리스 코어 |
| 8 | 캠페인/해금 시스템 | Progression | Vertical Slice | Not Started | — | 미션 조건 시스템 |
| 9 | 보상 화면 | UI | Vertical Slice | Not Started | — | 캠페인/해금 시스템, 씬 관리자 |
| 10 | 저장/불러오기 | Persistence | Vertical Slice | Not Started | — | 캠페인/해금 시스템 |
| 11 | 스킬 시스템 | Gameplay | Alpha | Not Started | — | 테트리스 코어, 게임 보드 |
| 12 | 메인 메뉴 | UI | Alpha | Not Started | — | 씬 관리자, 캠페인/해금 시스템 |
| 13 | 오디오 매니저 | Audio | Alpha | Not Started | — | — |
| 14 | 비주얼 이펙트 시스템 (inferred) | Polish | Full Vision | Not Started | — | 테트리스 코어, 스킬 시스템 |

---

## Categories

| Category | Description | Systems |
|----------|-------------|---------|
| **Core** | 모든 것이 의존하는 기반 시스템 | 게임 보드, 입력 시스템, 씬 관리자 |
| **Gameplay** | 게임을 재미있게 만드는 시스템 | 테트리스 코어, 미션 조건 시스템, 스킬 시스템 |
| **Progression** | 플레이어 성장과 진행 | 캠페인/해금 시스템 |
| **Persistence** | 저장과 연속성 | 저장/불러오기 |
| **UI** | 플레이어에게 정보 전달 | 게임 HUD, 게임 오버 화면, 보상 화면, 메인 메뉴 |
| **Audio** | 사운드 시스템 | 오디오 매니저 |
| **Polish** | 시각적 완성도 | 비주얼 이펙트 시스템 |

---

## Priority Tiers

| Tier | Definition | 해당 시스템 수 |
|------|------------|--------------|
| **MVP** | 코어 루프 테스트에 필요한 최소 시스템 | 7개 (1~7) |
| **Vertical Slice** | 완전한 1회 플레이 경험에 필요 | 3개 (8~10) |
| **Alpha** | 모든 기능이 러프하게 존재 | 3개 (11~13) |
| **Full Vision** | 폴리시 및 비주얼 완성 | 1개 (14) |

---

## Dependency Map

### Foundation Layer (의존 없음)

1. **게임 보드** — 10×20 그리드 데이터 구조, 모든 게임플레이의 기반
2. **입력 시스템** — 키보드 입력 처리, 모든 조작의 출발점
3. **씬 관리자** — 화면 전환 관리, 게임 흐름의 뼈대
4. **오디오 매니저** — 다른 시스템과 독립적으로 존재 가능

### Core Layer (Foundation에 의존)

1. **테트리스 코어** — 입력 시스템, 게임 보드
2. **게임 오버 화면** — 씬 관리자, 테트리스 코어

### Feature Layer (Core에 의존)

1. **미션 조건 시스템** ⚠️ — 테트리스 코어, 게임 보드
2. **스킬 시스템** ⚠️ — 테트리스 코어, 게임 보드
3. **캠페인/해금 시스템** — 미션 조건 시스템
4. **저장/불러오기** — 캠페인/해금 시스템

### Presentation Layer (Feature에 의존)

1. **게임 HUD** — 테트리스 코어, 미션 조건 시스템, 스킬 시스템
2. **보상 화면** — 캠페인/해금 시스템, 씬 관리자
3. **메인 메뉴** — 씬 관리자, 캠페인/해금 시스템

### Polish Layer

1. **비주얼 이펙트 시스템** — 테트리스 코어, 스킬 시스템

---

## Recommended Design Order

| Order | System | Priority | Layer | Est. Effort |
|-------|--------|----------|-------|-------------|
| 1 | 게임 보드 | MVP | Foundation | S |
| 2 | 입력 시스템 | MVP | Foundation | S |
| 3 | 씬 관리자 | MVP | Foundation | S |
| 4 | 테트리스 코어 | MVP | Core | M |
| 5 | 미션 조건 시스템 ⚠️ | MVP | Feature | M |
| 6 | 게임 HUD | MVP | Presentation | S |
| 7 | 게임 오버 화면 | MVP | Presentation | S |
| 8 | 캠페인/해금 시스템 | Vertical Slice | Feature | M |
| 9 | 보상 화면 | Vertical Slice | Presentation | S |
| 10 | 저장/불러오기 | Vertical Slice | Feature | S |
| 11 | 스킬 시스템 ⚠️ | Alpha | Feature | M |
| 12 | 메인 메뉴 | Alpha | Presentation | S |
| 13 | 오디오 매니저 | Alpha | Foundation | S |
| 14 | 비주얼 이펙트 시스템 | Full Vision | Polish | L |

*Effort: S = 1세션, M = 2~3세션, L = 4세션 이상*

---

## Circular Dependencies

없음 ✅

---

## High-Risk Systems

| System | Risk Type | Risk Description | Mitigation |
|--------|-----------|-----------------|------------|
| **미션 조건 시스템** | 설계 위험 | 어떤 조건 타입을 지원할지 결정이 게임의 다양성을 좌우. 너무 적으면 단조롭고, 너무 많으면 구현 복잡도 폭발 | 3가지 조건 타입으로 MVP 시작, 이후 점진적 추가 |
| **스킬 시스템** | 범위/밸런스 위험 | 스킬이 너무 강하면 미션 조건이 무의미해짐. 즉발+패시브 조합의 밸런스 검증 필요 | Alpha에서 프로토타입 후 플레이테스트로 밸런스 조정 |

---

## Art Direction

- **MVP ~ Alpha**: 단색 도형 (블록 = 색깔별 사각형)
- **Full Vision**: 3D 스타일 이펙트 추가 (Godot 4 셰이더 / 파티클 활용)

---

## Progress Tracker

| Metric | Count |
|--------|-------|
| Total systems identified | 14 |
| Design docs started | 5 |
| Design docs reviewed | 2 |
| Design docs approved | 2 |
| MVP systems designed | 5 / 7 |
| Vertical Slice systems designed | 0 / 3 |

---

## Next Steps

- [ ] `/design-system 게임-보드` — 첫 번째 시스템 GDD 작성
- [ ] `/design-system 테트리스-코어` — 핵심 게임플레이 GDD
- [ ] `/design-system 미션-조건-시스템` — 고위험 시스템 우선 설계
- [ ] `/prototype tetris-core` — 코어 루프 검증
- [ ] `/gate-check pre-production` — MVP 시스템 설계 완료 후 실행
