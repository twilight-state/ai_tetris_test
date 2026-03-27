# 게임 오버 화면 (Game Over Screen)

> **Status**: Designed
> **Author**: Luna/Solar Eclipse + Claude
> **Last Updated**: 2026-03-28
> **Implements Pillar**: 2분 안에 결판 (빠른 재도전 흐름으로 게임 템포 유지)

## Overview

게임 오버 화면은 테트리스 코어의 `game_over` 시그널(보드 꽉 참) 또는 미션 조건 시스템의 `mission_failed` 시그널(TIME_LIMIT 타이머 만료) 수신 시 표시되는 화면이다. 플레이어에게 종료 원인을 명확히 전달하고, 단일 행동(재시작)으로 즉시 다음 시도를 시작할 수 있는 경로를 제공한다. 씬 관리자를 통해 `Gameplay.tscn`으로 전환하며, 어떤 종료 원인이든 동일한 화면에서 처리한다. 이 시스템이 없으면 게임이 종료된 후 플레이어가 돌아올 방법이 없다.

## Player Fantasy

게임 오버의 순간은 짧고 명확해야 한다. "보드가 꽉 찼다" 또는 "시간이 다 됐다"는 사실을 인식하는 데 1초도 걸리지 않아야 하며, 재시작 버튼 하나로 다음 시도가 시작되어야 한다. 좌절감은 자연스럽지만, 그 좌절이 "한 번 더"라는 의지로 전환되는 속도가 이 화면의 품질을 결정한다. 긴 애니메이션, 복잡한 메뉴, 불필요한 통계 — 이 모든 것은 재도전의 흐름을 끊는다. Pillar 3("2분 안에 결판")의 빠른 템포는 게임 오버 이후에도 이어져야 한다. 플레이어가 화면을 읽는 시간보다 버튼을 누르는 시간이 더 짧아야 한다.

## Detailed Design

### Core Rules

1. **진입 트리거 (2가지)**
   - `game_over` 시그널 수신 (테트리스 코어 — 스폰 위치 충돌)
   - `mission_failed` 시그널 수신 (미션 조건 시스템 — TIME_LIMIT 타이머 만료)
   - 두 경우 모두 동일한 `GameOver.tscn`으로 전환. 원인은 `reason` 파라미터로 전달

2. **표시 정보**
   - 종료 원인 텍스트: `game_over` → "GAME OVER", `mission_failed` → "TIME OVER"
   - 현재 스테이지 번호 (씬 관리자로부터 전달받음)
   - 달성한 줄 수 또는 점수 (미션 조건 시스템으로부터 전달받음)
   - 재시작 버튼 1개

3. **재시작 동작**
   - 재시작 버튼 입력 시 `request_scene_change("Gameplay", {stage_id})` 호출
   - 동일 스테이지를 처음부터 재시작 (진행도 완전 리셋)
   - 씬 관리자가 페이드 아웃(0.2초) → `Gameplay.tscn` 로딩 → 페이드 인(0.2초) 처리

4. **입력 제한**
   - 화면 전환 애니메이션(페이드) 중에는 재시작 버튼 입력 무시
   - 화면이 완전히 표시된 후부터 버튼 입력 활성화

5. **자동 진행 없음**
   - 타이머 기반 자동 재시작 없음 — 반드시 플레이어가 버튼을 눌러야 전환
   - 무한 대기 허용

### States and Transitions

| 상태 | 진입 조건 | 탈출 조건 | 동작 |
|------|----------|----------|------|
| **FadeIn** | 씬 로딩 완료 | 페이드 인(0.2초) 완료 | UI 표시, 입력 비활성 |
| **Waiting** | 페이드 인 완료 | 재시작 버튼 입력 | 입력 활성, 버튼 대기 |
| **FadeOut** | 재시작 버튼 입력 | 페이드 아웃(0.2초) 완료 | 입력 비활성, 씬 전환 요청 |

### Interactions with Other Systems

| 시스템 | 방향 | 인터페이스 |
|--------|------|-----------|
| **테트리스 코어** | 코어 → 씬 관리자 → GameOver | `game_over` 시그널 → 씬 관리자가 `GameOver.tscn`으로 전환 |
| **미션 조건 시스템** | 미션 → 씬 관리자 → GameOver | `mission_failed` 시그널 → 씬 관리자가 `GameOver.tscn`으로 전환 |
| **씬 관리자** | GameOver → 씬 관리자 | 재시작 시 `request_scene_change("Gameplay", {stage_id})` 호출 |

## Formulas

게임 오버 화면은 자체 계산이 없다. 모든 표시값은 씬 전환 시 파라미터로 전달받는다.

```
# 씬 전환 시 전달되는 데이터
{
  reason: Enum { GAME_OVER, MISSION_FAILED },
  stage_id: int,
  current_value: int   // LINE_CLEAR·TIME_LIMIT → 클리어한 줄 수 / SCORE → 누적 점수
  target_value: int    // 미션 목표값 (달성 얼마나 근접했는지 표시용)
}
```

| 변수 | 타입 | 설명 |
|------|------|------|
| `reason` | Enum | 종료 원인. 표시 텍스트 결정에 사용 |
| `stage_id` | int | 현재 스테이지 번호 |
| `current_value` | int | 플레이어가 달성한 값 |
| `target_value` | int | 미션 목표값 |

## Edge Cases

| 시나리오 | 예상 동작 | 근거 |
|---------|----------|------|
| `game_over`와 `mission_failed`가 같은 프레임에 도달 | `game_over` 우선 처리 — "GAME OVER" 표시 | 보드 꽉 참이 더 명확한 종료 원인 |
| 페이드 인 중 재시작 버튼 입력 | 무시 — FadeIn 상태에서 입력 비활성 | 전환 완료 전 이중 전환 방지 |
| 씬 전환 데이터 없이 `GameOver.tscn` 로딩 | 기본값으로 표시 (reason=GAME_OVER, stage_id=1, values=0) | 직접 씬 로딩 방어 처리 |
| 재시작 버튼을 빠르게 여러 번 입력 | 첫 입력만 처리 — FadeOut 진입 후 추가 입력 무시 | 중복 씬 전환 방지 |

## Dependencies

| 시스템 | 방향 | 의존 성격 |
|--------|------|---------|
| **씬 관리자** | GameOver가 의존 | Hard — 씬 전환 데이터 수신 및 재시작 요청 모두 씬 관리자 경유 |
| **테트리스 코어** | GameOver가 간접 의존 | Hard — `game_over` 시그널이 이 화면의 진입 트리거 중 하나 |
| **미션 조건 시스템** | GameOver가 간접 의존 | Hard — `mission_failed` 시그널이 이 화면의 진입 트리거 중 하나 |

> 씬 관리자 GDD 교차 확인: "게임 오버 화면 → 씬 관리자, 재시작 선택 시 `request_scene_change("Gameplay")` 호출" ✅
> 미션 조건 시스템 GDD Open Questions: "`mission_failed` 시 즉시 게임 오버인가?" → 이 문서에서 결정: **즉시 GameOver.tscn 전환** ✅

## Tuning Knobs

| 파라미터 | 현재 값 | 안전 범위 | 높이면 | 낮추면 |
|---------|--------|---------|--------|--------|
| 페이드 인/아웃 시간 | 0.2초 | 0.1 ~ 0.5초 | 전환이 부드럽지만 느리게 느껴짐 | 너무 빠르면 화면 전환을 인지 못함 |

> 페이드 시간은 씬 관리자 GDD 소유값과 동일. 게임 오버 화면 단독으로 변경 불가 — 씬 관리자 Tuning Knobs에서 조정.

## Visual/Audio Requirements

| 이벤트 | 시각 피드백 | 오디오 피드백 | 우선순위 |
|--------|-----------|-------------|---------|
| 화면 진입 (FadeIn) | 페이드 인 0.2초 | 없음 | High |
| 종료 원인 표시 | "GAME OVER" 또는 "TIME OVER" 텍스트 중앙 표시 | 게임 오버 효과음 (1회) | High |
| 진행도 표시 | `current_value / target_value` 텍스트 | 없음 | Medium |
| 재시작 버튼 호버 | 버튼 색상 변화 | 없음 (MVP) | Low |
| 재시작 입력 (FadeOut) | 페이드 아웃 0.2초 | 없음 | High |

> MVP: 단색 + 기본 폰트. 오디오는 게임 오버 효과음 1종만.

## UI Requirements

| 정보 | 표시 위치 | 갱신 주기 | 조건 |
|------|---------|---------|------|
| 종료 원인 텍스트 ("GAME OVER" / "TIME OVER") | 화면 중앙 상단 | 씬 로딩 시 1회 | 항상 |
| 스테이지 번호 | 종료 텍스트 아래 | 씬 로딩 시 1회 | 항상 |
| 진행도 수치 (`current / target`) | 스테이지 번호 아래 | 씬 로딩 시 1회 | 항상 |
| 재시작 버튼 | 화면 중앙 하단 | — | Waiting 상태 |

## Acceptance Criteria

**진입**
- [ ] 테트리스 코어에서 `game_over` 시그널 발신 시 `GameOver.tscn`으로 전환된다
- [ ] 미션 조건 시스템에서 `mission_failed` 시그널 발신 시 `GameOver.tscn`으로 전환된다
- [ ] `game_over` 진입 시 "GAME OVER" 텍스트가 표시된다
- [ ] `mission_failed` 진입 시 "TIME OVER" 텍스트가 표시된다

**표시**
- [ ] 스테이지 번호가 올바르게 표시된다
- [ ] `current_value / target_value` 진행도 수치가 올바르게 표시된다

**재시작**
- [ ] 재시작 버튼 입력 시 동일 스테이지의 `Gameplay.tscn`으로 전환된다
- [ ] 페이드 인 중 재시작 버튼 입력이 무시된다
- [ ] 재시작 버튼을 빠르게 여러 번 눌러도 씬 전환이 1회만 발생한다

**엣지 케이스**
- [ ] `game_over`와 `mission_failed`가 같은 프레임에 도달하면 "GAME OVER"가 표시된다
- [ ] 씬 전환 데이터 없이 로딩 시 기본값으로 표시되고 크래시가 없다

## Open Questions

| 질문 | 담당 | 해결 시점 |
|------|------|---------|
| 게임 오버 화면에서 메인 메뉴로 돌아가는 버튼이 MVP에 필요한가? | 스코프 결정 | Alpha 단계 (메인 메뉴 추가 시) |
| 진행도 수치 외에 추가 통계(최고 기록 등)를 표시할 것인가? | Vertical Slice 이후 결정 | Alpha 단계 |
