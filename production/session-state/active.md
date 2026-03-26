# Session State

**Last Updated**: 2026-03-27
**Current Task**: 씬 관리자 GDD 설계 (다음 예정)
**Status**: 입력 시스템 GDD 완료

## Progress Checklist

- [x] 게임 컨셉 문서 (design/gdd/game-concept.md)
- [x] 시스템 인덱스 (design/gdd/systems-index.md)
- [x] 게임 보드 GDD (design/gdd/game-board.md) — Designed
- [x] 입력 시스템 GDD (design/gdd/input-system.md) — Approved
  - [x] Overview
  - [x] Player Fantasy
  - [x] Detailed Design (Core Rules, States, Interactions)
  - [x] Formulas (DAS/ARR, SDF)
  - [x] Edge Cases
  - [x] Dependencies
  - [x] Tuning Knobs
  - [x] Acceptance Criteria
  - [x] Open Questions
- [ ] 씬 관리자 GDD — 다음 작업

## Key Decisions (입력 시스템)

- 키보드 전용 (화살표 + WASD 듀얼 서포트)
- DAS: 167ms / ARR: 33ms / SDF: 15x (Tetris Guideline 기준값)
- ARR=0 지원 (즉시 이동)
- Hold 기능 포함 (C/Shift)
- 버퍼링 없음
- 시그널 전용 아키텍처 — 직접 시스템 호출 없음
- 상태 전환 트리거 소유권: 씬 관리자

## Open Questions (미결)

- Frozen 상태 중 입력 처리 → 스킬 시스템 GDD에서 결정
- DAS/ARR 인게임 설정 UI → 게임 HUD GDD에서 결정

## MVP 진행률

게임 보드 [✅ Designed] | 입력 시스템 [✅ Approved] | 씬 관리자 [ ] | 테트리스 코어 [ ] | 미션 조건 시스템 [ ] | 게임 HUD [ ] | 게임 오버 화면 [ ]
