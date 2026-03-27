# 미션 조건 시스템 (Mission Condition System)

> **Status**: In Design
> **Author**: Luna/Solar Eclipse + Claude
> **Last Updated**: 2026-03-28
> **Implements Pillar**: 매 스테이지가 새로운 퍼즐 (이 시스템이 "새로운 퍼즐"을 정의)

## Overview

미션 조건 시스템은 스테이지마다 다른 클리어 조건을 정의하고, 게임플레이 중 그 달성 여부를 실시간으로 추적하는 시스템이다. 테트리스 코어의 이벤트(라인 클리어, 블록 고정, 하드 드롭 등)와 게임 보드의 상태를 구독하여 진행도를 갱신하며, 조건 달성 또는 실패 시 씬 관리자·캠페인 시스템·게임 HUD에 시그널을 발신한다. MVP에서는 3가지 조건 타입을 지원하며, 각 스테이지의 조건은 외부 데이터(Resource/JSON)로 정의되어 코드 변경 없이 새 스테이지를 추가할 수 있다. 이 시스템이 없으면 모든 스테이지가 "그냥 테트리스"가 된다.

## Player Fantasy

미션 조건 시스템은 "이번엔 다른 방식으로 해야 한다"는 인식의 순간을 만든다. 스테이지 시작 시 조건을 확인하고 전략을 세우는 짧은 계획의 즐거움, 그리고 어렵게만 느껴지던 조건이 마침내 충족되는 순간의 폭발적인 성취감이 이 시스템이 줘야 할 핵심 감정이다. Pillar 1("매 스테이지가 새로운 퍼즐")의 실체는 바로 이 시스템이 정의하는 조건이다. 조건은 도전이어야 하지만 불가능처럼 느껴지면 안 된다 — "어렵다"에서 시작해 "됐다!"로 끝나는 곡선이 매 스테이지에서 반복되어야 한다. Tetris Effect의 라인 클리어 순간처럼, 조건 달성의 순간에는 시각/오디오 피드백이 그 쾌감을 극대화해야 한다.

## Detailed Design

### Core Rules

**1. 미션 데이터 구조**

각 스테이지는 `MissionData` Resource로 조건을 정의한다:

```
MissionData:
  condition_type: Enum { LINE_CLEAR, SCORE, TIME_LIMIT }
  target: int        // LINE_CLEAR: 목표 줄 수 / SCORE: 목표 점수 / TIME_LIMIT: 목표 줄 수
  time_limit: float  // TIME_LIMIT 전용 (초). 다른 타입에서는 0 (사용 안 함)
  stage_id: int      // 어느 스테이지의 조건인지 식별
```

**2. 조건 타입 동작**

| 타입 | 달성 조건 | 실패 조건 | 트래킹 소스 |
|------|----------|----------|-----------|
| **LINE_CLEAR** | `current_lines >= target` | 게임 오버 (외부 처리) | `line_cleared(count)` 시그널 |
| **SCORE** | `current_score >= target` | 게임 오버 (외부 처리) | `line_cleared(count)` + 점수 계산 |
| **TIME_LIMIT** | `current_lines >= target` (시간 내) | 타이머 만료 또는 게임 오버 | `line_cleared(count)` + `_process()` 타이머 |

**3. 이벤트 처리**
- 테트리스 코어의 `line_cleared(count: int)` 시그널 구독
- `_process(delta)`에서 TIME_LIMIT 타이머 카운트다운 (Active 상태이고 TIME_LIMIT 타입일 때만)
- 상태 변화 시마다 HUD 갱신 시그널 발신

**4. 출력 시그널**
- `progress_updated(current: int, target: int)` — HUD 진행도 갱신
- `time_updated(remaining: float)` — TIME_LIMIT HUD 타이머 갱신
- `mission_complete` — 씬 관리자 + 캠페인 시스템에 클리어 알림
- `mission_failed` — TIME_LIMIT 타이머 만료 시 (게임 오버와 별개)

**5. 초기화 및 활성화**
- 스테이지 시작 시 `load_mission(MissionData)` 호출 → 모든 트래커 리셋
- `activate()` 호출로 시그널 구독 시작
- `deactivate()` 호출로 구독 해제 (게임 오버, 클리어, 씬 전환 시)

### States and Transitions

| 상태 | 진입 조건 | 탈출 조건 | 동작 |
|------|----------|----------|------|
| **Inactive** | 시스템 초기화 / 스테이지 종료 | `activate()` 호출 | 모든 이벤트 무시 |
| **Active** | `activate()` 호출 | 달성 조건 충족 또는 실패 | 이벤트 수신, 진행도 추적, HUD 갱신 |
| **Complete** | 달성 조건 충족 | `deactivate()` 호출 | `mission_complete` 발신, 추가 이벤트 무시 |
| **Failed** | TIME_LIMIT 타이머 만료 | `deactivate()` 호출 | `mission_failed` 발신 |

> 게임 오버는 테트리스 코어가 `game_over` 시그널로 처리. 씬 관리자가 `deactivate()` 호출로 정리.

### Interactions with Other Systems

| 시스템 | 방향 | 인터페이스 |
|--------|------|-----------|
| **테트리스 코어** | 코어 → 미션 | 시그널 수신: `line_cleared(count: int)` |
| **게임 보드** | 미션 → 보드 | 읽기 전용 — MVP에서는 시그널로 충분, 직접 보드 읽기 미사용 |
| **씬 관리자** | 미션 → 씬 | 시그널 발신: `mission_complete`, `mission_failed` → 다음 씬 결정 |
| **캠페인/해금 시스템** | 미션 → 캠페인 | 시그널 발신: `mission_complete` → 클리어 기록 및 보상 결정 |
| **게임 HUD** | 미션 → HUD | 시그널 발신: `progress_updated(current: int, target: int)`, `time_updated(remaining: float)` |

## Formulas

### LINE_CLEAR 진행도

```
on line_cleared(count):
    current_lines += count
    emit progress_updated(current_lines, target)
    if current_lines >= target:
        → Complete 상태
```

| 변수 | 타입 | 범위 | 설명 |
|------|------|------|------|
| `current_lines` | int | 0 ~ target | 현재까지 클리어한 줄 수 |
| `target` | int | 1 ~ 40 | 클리어 목표 줄 수 (MissionData에서 로드) |

### SCORE 점수 계산

```
on line_cleared(count):
    current_score += score_table[count]
    emit progress_updated(current_score, target)
    if current_score >= target:
        → Complete 상태
```

**점수 테이블 (표준 테트리스 가이드라인)**

| 클리어 줄 수 | 점수 |
|------------|------|
| 1 | 100 |
| 2 | 300 |
| 3 | 500 |
| 4 (Tetris!) | 800 |

| 변수 | 타입 | 범위 | 설명 |
|------|------|------|------|
| `current_score` | int | 0 ~ target | 현재 누적 점수 |
| `target` | int | 100 ~ 10000 | 목표 점수 (MissionData에서 로드) |
| `score_table` | Dictionary | — | 클리어 줄 수 → 점수 매핑 (외부 데이터로 관리) |

### TIME_LIMIT 타이머 및 진행도

```
on activate() (TIME_LIMIT 타입):
    remaining_time = time_limit

_process(delta) (TIME_LIMIT 타입, Active 상태):
    remaining_time -= delta
    emit time_updated(remaining_time)
    if remaining_time <= 0:
        → Failed 상태
        emit mission_failed

on line_cleared(count) (TIME_LIMIT 타입):
    current_lines += count
    emit progress_updated(current_lines, target)
    if current_lines >= target:
        → Complete 상태  // 타이머가 남아있어도 줄 수 달성 시 즉시 클리어
```

| 변수 | 타입 | 범위 | 설명 |
|------|------|------|------|
| `remaining_time` | float | 0.0 ~ time_limit | 남은 시간 (초) |
| `time_limit` | float | 10.0 ~ 300.0 | 스테이지 시간 제한 (MissionData에서 로드) |

> **튜닝 주의**: 스테이지 낙하 속도(`fall_interval`)와 `time_limit`은 연동해서 설정해야 한다.
> 낙하가 빠를수록 `time_limit`을 넉넉히 설정해야 클리어 가능성이 유지된다.

## Edge Cases

| 시나리오 | 예상 동작 | 근거 |
|---------|----------|------|
| 같은 프레임에 LINE_CLEAR 달성과 게임 오버가 동시 발생 | `mission_complete` 우선 발신 — 클리어로 처리 | 플레이어에게 유리한 판정 원칙 |
| TIME_LIMIT: 마지막 라인 클리어로 목표 달성과 타이머 만료가 동시 | `mission_complete` 우선 발신 — 클리어로 처리 | 위와 동일 원칙 |
| `line_cleared(count=4)` 수신 시 SCORE가 목표를 초과 | 초과 무관 — `current_score >= target` 조건으로 Complete | 오버슈트 허용, 초과분 버림 |
| `load_mission()` 없이 `activate()` 호출 | assert 실패 또는 Inactive 유지 — 조건 없는 활성화 불가 | 데이터 무결성 보장 |
| Active 상태에서 `load_mission()` 재호출 | `deactivate()` 후 새 데이터로 재초기화 — 이전 진행도 완전 리셋 | 스테이지 재시작 처리 |
| LINE_CLEAR / SCORE: `target = 0` 설정 | assert 실패 — 데이터 검증으로 방지 | 즉시 클리어되는 의미 없는 조건 방지 |
| TIME_LIMIT 타입에서 `time_limit = 0` 설정 | assert 실패 — TIME_LIMIT에는 반드시 양수 time_limit 필요 | 의미 없는 조건 방지 |
| 게임 오버 후 `line_cleared` 시그널이 늦게 도달 | Inactive 전환 후 시그널 무시 | `deactivate()` 즉시 시그널 구독 해제로 자동 방지 |
| 일시정지 중 TIME_LIMIT 타이머 | 일시정지 중 `_process()` 멈춤 — 자동 정지 | 씬 관리자 GDD: `SceneTree.paused = true` 적용 ✅ |

## Dependencies

| 시스템 | 방향 | 의존 성격 |
|--------|------|---------|
| **테트리스 코어** | 미션이 의존 | Hard — `line_cleared` 시그널 없이 진행도 추적 불가 |
| **게임 보드** | 미션이 의존 | Soft — MVP에서는 직접 보드 읽기 미사용. 향후 확장 조건 타입에서 필요 가능 |
| **캠페인/해금 시스템** | 캠페인이 이 시스템에 의존 | Hard — `mission_complete` 시그널로 클리어 기록 및 보상 결정 |
| **게임 HUD** | HUD가 이 시스템에 의존 | Hard — `progress_updated`, `time_updated` 시그널 없이 진행도 UI 불가 |
| **씬 관리자** | 씬이 이 시스템에 의존 | Hard — `mission_complete`, `mission_failed` 시그널로 씬 전환 결정 |

> 테트리스 코어 GDD 교차 확인: "미션이 이 시스템에 의존 — Hard, `line_cleared` 시그널" ✅
> 게임 보드 GDD 교차 확인: "미션이 이 시스템에 의존 — Hard, 읽기 전용" ✅

## Tuning Knobs

[To be designed]

## Visual/Audio Requirements

[To be designed]

## UI Requirements

[To be designed]

## Acceptance Criteria

[To be designed]

## Open Questions

[To be designed]
