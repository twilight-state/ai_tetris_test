# Session State

<!-- STATUS -->
Epic: 블록 원정대
Feature: Pre-Production — System GDDs
Task: 입력 시스템 GDD 작성 (MVP Foundation #2)
<!-- /STATUS -->

## Current Task
게임 보드 GDD 완료. 다음: 입력 시스템 GDD 작성

## Progress Checklist
- [x] 게임 컨셉 문서 (design/gdd/game-concept.md)
- [x] 엔진 설정 — Godot 4.6.1
- [x] 시스템 인덱스 (design/gdd/systems-index.md)
- [x] 게임 보드 GDD (design/gdd/game-board.md) — MVP 1/7
- [ ] 입력 시스템 GDD ← 다음
- [ ] 씬 관리자 GDD
- [ ] 테트리스 코어 GDD
- [ ] 미션 조건 시스템 GDD (고위험)
- [ ] 게임 HUD GDD
- [ ] 게임 오버 화면 GDD

## Key Decisions
- 엔진: Godot 4.6.1
- 보드: 10x20 + 3행 스폰 버퍼, 셀 = {type_id, color}
- 아트: MVP는 단색 도형, Full Vision에서 3D 이펙트
- 스테이지: 10개 선형 캠페인

## Next Step
/design-system 입력-시스템
