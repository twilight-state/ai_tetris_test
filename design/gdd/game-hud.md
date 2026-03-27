# 게임 HUD (Game HUD)

> **Status**: Designed
> **Author**: Luna/Solar Eclipse + Claude
> **Last Updated**: 2026-03-28
> **Implements Pillar**: 피드백 명확성 (플레이어가 미션 진행도와 게임 상태를 한눈에 파악)

## Overview

게임 HUD는 게임플레이 화면에서 플레이어가 의사결정에 필요한 정보를 실시간으로 표시하는 시스템이다. 테트리스 코어로부터 활성 피스·Ghost Piece·Hold 슬롯·Next 프리뷰를 수신해 렌더링하고, 미션 조건 시스템으로부터 진행도·남은 시간을 수신해 표시한다. HUD는 직접 게임 상태를 계산하지 않으며 — 모든 데이터는 시그널로 수신한다. 이 시스템이 없으면 플레이어는 다음 블록이 무엇인지, 미션 달성에 얼마나 근접했는지 알 수 없다.

## Player Fantasy

HUD는 플레이어가 의식하지 않아야 하지만, 없으면 즉시 느껴지는 시스템이다. "다음 블록이 I피스다 — 여기에 끼워넣으면 테트리스가 된다"는 전략적 판단이 0.5초 안에 이루어질 수 있는 것은 Next 프리뷰 덕분이다. 미션 진행도 바가 서서히 차오를 때의 기대감, 남은 시간이 5초 이하로 떨어지면서 타이머가 빨갛게 변할 때의 극도의 집중 — 이 감각적 정보들이 플레이어를 Flow 상태로 유지시킨다. 게임 컨셉의 "피드백 명확성: 미션 진행도 UI, 스킬 쿨다운 표시"가 이 시스템의 존재 이유다.

## Detailed Design

### Core Rules

1. **HUD는 계산하지 않는다**
   - 모든 데이터는 시그널로 수신. HUD는 수신한 값을 그대로 렌더링만 한다
   - 게임 상태(점수, 줄 수, 진행도)를 직접 계산하거나 저장하지 않음

2. **활성 피스 렌더링**
   - `active_piece_changed` 수신 시 피스의 현재 좌표·회전 상태를 보드 위에 즉시 반영
   - Active / Locking 상태에서만 렌더링 (Inactive·GameOver 상태에서는 숨김)

3. **Ghost Piece 렌더링**
   - `active_piece_changed` 수신 시 Ghost 위치를 동시에 갱신 (Ghost 위치 계산은 테트리스 코어가 시그널에 포함해 전달)
   - Ghost와 Active Piece가 같은 위치면 Ghost 표시 안 함 (이미 최하단)
   - **MVP 포함 여부**: Open Questions 참조

4. **Hold 슬롯**
   - 보드 좌측 상단 고정 위치. `hold_changed` 수신 시 갱신
   - Hold 슬롯 비어있으면 빈 박스 표시 (스테이지 시작 초기 상태)
   - Hold 쿨다운 활성 중: 피스 이미지 위에 반투명 어둡게 오버레이 → 쿨다운 해제 시 오버레이 제거

5. **Next 프리뷰**
   - 보드 우측 상단 고정 위치. `next_changed` 수신 시 갱신
   - 항상 1개만 표시. 슬롯 항상 채워진 상태 (스테이지 시작 시점부터)

6. **미션 진행도 바**
   - `progress_updated(current, target)` 수신 시 `current / target` 비율로 바 길이 갱신
   - 진행도 바 위에 조건 설명 텍스트 고정 표시 (예: "10줄 클리어", "1000점 달성"). `load_mission()` 시 1회 셋팅, 이후 불변
   - `current / target` 수치 텍스트도 바 옆에 표시 (예: "7 / 10")

7. **타이머 (TIME_LIMIT 전용)**
   - `time_updated(remaining)` 수신 시 `ceil(remaining)` 값을 초 단위 정수로 텍스트 갱신
   - `remaining > 5.0`: 기본 색상 (흰색)
   - `remaining <= 5.0`: 빨간색으로 전환 + 0.5초 주기 깜박임 시작
   - TIME_LIMIT 타입이 아닌 스테이지에서는 타이머 UI 전체 숨김

8. **라인 클리어 팝업**
   - `line_cleared(count)` 수신 시 count에 따라 텍스트 0.8초간 표시 후 자동 숨김
   - 매핑: `1 → "SINGLE"`, `2 → "DOUBLE"`, `3 → "TRIPLE"`, `4 → "TETRIS!"`
   - 팝업이 표시 중에 추가 클리어 발생하면 이전 팝업 즉시 교체 (타이머 리셋)
   - 보드 중앙 상단 오버레이 표시

### States and Transitions

| 상태 | 진입 조건 | 탈출 조건 | 동작 |
|------|----------|----------|------|
| **Inactive** | 초기화 / 스테이지 종료 | `setup(MissionData)` + `activate()` 호출 | 모든 UI 숨김 또는 비활성 표시 |
| **Active** | `activate()` 호출 | `deactivate()` 호출 | 시그널 수신 및 UI 갱신 |
| **Paused** | 씬 관리자 일시정지 | 일시정지 해제 | UI 동결 (타이머 깜박임 포함 중단). SceneTree.paused = true 적용으로 자동 처리 |

> 씬 관리자가 `SceneTree.paused = true`를 사용하므로 Paused 상태는 별도 로직 없이 자동으로 _process() 정지.

### Interactions with Other Systems

| 시스템 | 방향 | 인터페이스 |
|--------|------|-----------|
| **테트리스 코어** | 코어 → HUD | 시그널 수신: `active_piece_changed(piece_data)`, `hold_changed(hold_data)`, `next_changed(next_type)`, `line_cleared(count: int)` |
| **미션 조건 시스템** | 미션 → HUD | 시그널 수신: `progress_updated(current: int, target: int)`, `time_updated(remaining: float)` |
| **씬 관리자** | 씬 → HUD | `setup(MissionData)` 호출로 조건 타입·텍스트 초기화, `activate()` / `deactivate()` 호출 |

## Formulas

### 진행도 바 비율

```
bar_fill_ratio = clamp(current / float(target), 0.0, 1.0)
```

| 변수 | 타입 | 범위 | 설명 |
|------|------|------|------|
| `current` | int | 0 ~ target | 미션 조건 시스템에서 수신한 현재 진행값 |
| `target` | int | 1 ~ (조건 타입별 최대값) | 미션 조건 시스템에서 수신한 목표값 |
| `bar_fill_ratio` | float | 0.0 ~ 1.0 | 진행도 바 길이 비율. `clamp`로 초과 방지 |

> `target`이 0인 경우는 미션 조건 시스템 GDD에서 assert로 방지 — HUD에서 추가 방어 불필요.

### 타이머 표시 변환

```
display_seconds = ceil(remaining_time)
```

| 변수 | 타입 | 범위 | 설명 |
|------|------|------|------|
| `remaining_time` | float | 0.0 ~ time_limit | 미션 조건 시스템에서 수신한 남은 시간 (초) |
| `display_seconds` | int | 0 ~ ceil(time_limit) | 표시용 올림 정수. 0.1초가 남아도 "1"로 표시해 플레이어에게 유리 |

### 타이머 경고 임계값

```
is_warning = (remaining_time <= WARNING_THRESHOLD)
```

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `WARNING_THRESHOLD` | 5.0초 | 이 값 이하에서 타이머 빨간색 + 깜박임 활성화 |

### 라인 클리어 팝업 지속 시간

```
popup_timer = POPUP_DURATION  // 팝업 표시 시 리셋
popup_timer -= delta           // 매 프레임 감소
if popup_timer <= 0: 팝업 숨김
```

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `POPUP_DURATION` | 0.8초 | 라인 클리어 팝업이 화면에 머무는 시간 |

## Edge Cases

| 시나리오 | 예상 동작 | 근거 |
|---------|----------|------|
| Ghost와 Active Piece가 같은 위치 (이미 최하단) | Ghost 숨김 | 시각적 혼란 방지 |
| Hold 슬롯 비어있을 때 `hold_changed` 수신 | 빈 박스 렌더링 유지 (데이터 없음 표시) | 슬롯은 항상 표시 |
| TIME_LIMIT 아닌 스테이지에서 `time_updated` 수신 | 무시 — 타이머 UI 숨긴 상태 유지 | 조건 타입과 무관한 시그널이 도달할 수 있음 |
| 팝업 표시 중 추가 `line_cleared` 수신 | 기존 팝업 즉시 교체, `popup_timer` 리셋 | 연속 클리어 시 최신 정보 우선 |
| `activate()` 없이 시그널 수신 | 무시 — Inactive 상태에서 렌더링 없음 | 상태 전환 테이블 준수 |
| `progress_updated`에서 `current > target` | `bar_fill_ratio = 1.0` (clamp로 자동 처리) | 초과 진행도는 꽉 찬 바로 표시 |
| 스테이지 종료 순간 팝업 표시 중 | `deactivate()` 호출 시 팝업 포함 모든 UI 즉시 숨김 | 씬 전환 시 HUD 잔상 방지 |

## Dependencies

| 시스템 | 방향 | 의존 성격 |
|--------|------|---------|
| **테트리스 코어** | HUD가 의존 | Hard — `active_piece_changed`, `hold_changed`, `next_changed`, `line_cleared` 없이 보드 측 UI 렌더링 불가 |
| **미션 조건 시스템** | HUD가 의존 | Hard — `progress_updated`, `time_updated` 없이 미션 진행도·타이머 표시 불가 |
| **씬 관리자** | HUD가 의존 | Hard — `setup()` / `activate()` / `deactivate()` 호출로 HUD 생명주기 제어 |
| **게임 보드** | 의존 없음 | 없음 — HUD는 보드를 직접 읽지 않음. 모든 정보는 시그널로 수신 |

> 테트리스 코어 GDD 교차 확인: "HUD가 이 시스템에 의존 — Hard" ✅
> 미션 조건 시스템 GDD 교차 확인: "HUD가 이 시스템에 의존 — Hard" ✅

## Tuning Knobs

| 파라미터 | 현재 값 | 안전 범위 | 높이면 | 낮추면 |
|---------|--------|---------|--------|--------|
| `WARNING_THRESHOLD` | 5.0초 | 3.0 ~ 10.0초 | 경고가 일찍 시작 — 여유가 있어도 압박감 증가 | 경고가 너무 늦게 시작 — 반응 시간 부족 |
| `POPUP_DURATION` | 0.8초 | 0.3 ~ 1.5초 | 팝업이 오래 남아 시야 방해 | 너무 짧아 텍스트를 못 읽음 |
| 타이머 깜박임 주기 | 0.5초 | 0.2 ~ 1.0초 | 느린 깜박임 — 긴박감 약함 | 너무 빠른 깜박임 — 시각적 피로 |
| Hold 쿨다운 오버레이 불투명도 | 0.5 (50%) | 0.3 ~ 0.8 | 쿨다운 강조 강해짐 — 피스 이미지가 안 보임 | 쿨다운 활성 여부 구분 어려움 |

## Visual/Audio Requirements

| 이벤트 | 시각 피드백 | 오디오 피드백 | 우선순위 |
|--------|-----------|-------------|---------|
| 활성 피스 이동/회전 | 피스 위치·회전 즉시 갱신, Ghost 위치 동시 갱신 | 없음 (MVP) | High |
| Hold 변경 | Hold 슬롯 즉시 갱신 | 없음 (MVP) | High |
| Hold 쿨다운 활성 | 피스 이미지 위 반투명 오버레이 | 없음 (MVP) | High |
| Next 변경 | Next 슬롯 즉시 갱신 | 없음 (MVP) | High |
| 라인 클리어 팝업 | "SINGLE" / "DOUBLE" / "TRIPLE" / "TETRIS!" 텍스트 0.8초 표시 | 없음 (MVP, 효과음은 테트리스 코어 GDD에서 처리) | Medium |
| 미션 진행도 갱신 | 진행도 바 길이 애니메이션 | 없음 (MVP) | Medium |
| 타이머 경고 (5초 이하) | 타이머 텍스트 빨간색 전환 + 0.5초 주기 깜박임 | 없음 (MVP) | High |
| `deactivate()` 호출 | 모든 HUD UI 즉시 숨김 | 없음 | High |

> MVP: 모든 UI 요소는 단색 도형 + 기본 폰트로 구현. 오디오는 테트리스 코어 GDD에서 담당하며 HUD는 오디오 처리 없음.

## UI Requirements

| 정보 | 표시 위치 | 갱신 주기 | 조건 |
|------|---------|---------|------|
| 활성 피스 | 보드 위 (좌표 기반 렌더링) | `active_piece_changed` 수신 시 | Active / Locking 상태 |
| Ghost Piece | 보드 위 (반투명, 활성 피스 아래) | `active_piece_changed` 수신 시 | Active / Locking 상태, Ghost ≠ Active 위치 |
| Hold 슬롯 | 보드 좌측 상단 고정 | `hold_changed` 수신 시 | 항상 표시 (비어있으면 빈 박스) |
| Hold 쿨다운 오버레이 | Hold 슬롯 위 | Hold 쿨다운 변경 시 | 쿨다운 활성 중만 표시 |
| Next 프리뷰 | 보드 우측 상단 고정 | `next_changed` 수신 시 | 항상 표시 |
| 미션 조건 설명 텍스트 | 진행도 바 상단 | `setup()` 시 1회 | Active 상태 |
| 미션 진행도 바 | 보드 우측 또는 하단 | `progress_updated` 수신 시 | Active 상태 |
| 진행도 수치 텍스트 | 진행도 바 옆 | `progress_updated` 수신 시 | Active 상태 |
| 타이머 | 보드 상단 중앙 | `time_updated` 수신 시 | TIME_LIMIT 타입 + Active 상태 |
| 타이머 경고 스타일 | 타이머 텍스트 | `remaining <= WARNING_THRESHOLD` 시 | TIME_LIMIT 타입 |
| 라인 클리어 팝업 | 보드 중앙 상단 오버레이 | `line_cleared` 수신 시 | 0.8초간 표시 후 자동 숨김 |

## Acceptance Criteria

**활성 피스 / Ghost Piece**
- [ ] `active_piece_changed` 수신 시 피스 위치·회전이 즉시 갱신된다
- [ ] Ghost Piece가 활성 피스 바로 아래 최하단 위치에 표시된다
- [ ] Ghost와 Active가 같은 위치이면 Ghost가 표시되지 않는다

**Hold / Next**
- [ ] Hold 슬롯이 비어있으면 빈 박스가 표시된다
- [ ] `hold_changed` 수신 시 Hold 슬롯이 즉시 갱신된다
- [ ] Hold 쿨다운 활성 중에 슬롯 위에 오버레이가 표시되고, 해제 시 사라진다
- [ ] `next_changed` 수신 시 Next 프리뷰가 즉시 갱신된다

**미션 진행도**
- [ ] `progress_updated(3, 10)` 수신 시 진행도 바가 30% 길이로 갱신된다
- [ ] `progress_updated(12, 10)` 수신 시 진행도 바가 100%로 표시된다 (초과 clamp)
- [ ] 미션 조건 설명 텍스트가 `setup()` 호출 후 변경되지 않는다

**타이머**
- [ ] TIME_LIMIT 타입에서만 타이머 UI가 표시된다
- [ ] `time_updated(5.5)` 수신 시 "6"이 표시된다 (ceil)
- [ ] `remaining <= 5.0`이면 빨간색 + 깜박임이 시작된다
- [ ] TIME_LIMIT 아닌 스테이지에서 `time_updated`가 수신돼도 타이머 UI가 표시되지 않는다

**라인 클리어 팝업**
- [ ] `line_cleared(1/2/3/4)` 수신 시 각각 "SINGLE"/"DOUBLE"/"TRIPLE"/"TETRIS!" 팝업이 표시된다
- [ ] 팝업이 0.8초 후 자동으로 사라진다
- [ ] 팝업 표시 중 추가 `line_cleared` 수신 시 팝업이 즉시 교체된다

**생명주기**
- [ ] `activate()` 전에는 모든 UI가 숨겨진다
- [ ] `deactivate()` 호출 시 팝업 포함 모든 UI가 즉시 숨겨진다

## Open Questions

| 질문 | 담당 | 해결 시점 |
|------|------|---------|
| Ghost Piece를 MVP에 포함할 것인가? | 스코프 결정 | 구현 시작 전 |
| 미션 진행도 바를 보드 우측에 둘 것인가, 하단에 둘 것인가? (레이아웃 최종 결정) | UI 레이아웃 검토 시 | 프로토타입 단계 |
| 라인 클리어 팝업 텍스트를 한글로 표시할 것인가? ("테트리스!" vs "TETRIS!") | 로컬라이제이션 방향 결정 시 | Alpha 단계 |
