# 씬 관리자 (Scene Manager)

> **Status**: In Design
> **Author**: Luna/Solar Eclipse + Claude
> **Last Updated**: 2026-03-27
> **Implements Pillar**: 2분 안에 결판 (빠른 씬 전환이 게임 흐름의 끊김을 방지)

## Overview

씬 관리자는 게임 내 모든 화면 전환을 단일 진입점으로 처리하는 시스템이다. 메인 메뉴 → 게임플레이 → 게임 오버 → 보상 화면 등의 씬 이동을 관리하며, 전환 시 입력 시스템의 상태(Active/Inactive)를 함께 조정한다. 씬 관리자는 "어느 화면으로 이동할지"만 결정하며, 각 씬 내부의 게임 상태(점수, 미션 진행도 등)는 해당 씬의 시스템이 직접 관리한다. `pause_toggled` 시그널을 수신해 일시정지 화면 전환도 처리한다.

## Player Fantasy

씬 관리자는 플레이어가 직접 인식하지 않아야 하는 시스템이다. 성공의 기준은 "화면이 자연스럽게 바뀌었다"는 느낌이다. 게임 오버 후 즉시 재시작 버튼을 눌렀을 때 끊김 없이 다음 화면으로 이어지고, 클리어 시에는 짧은 페이드 인/아웃으로 성취감을 방해하지 않으면서도 전환을 알린다. Pillar 3("2분 안에 결판")의 빠른 템포를 유지하려면 씬 전환이 게임 흐름의 방해물이 되어서는 안 된다. 전환 자체는 인식되지 않고 결과(다음 화면)만 남아야 한다.

## Detailed Design

### Core Rules

1. **씬 목록 (MVP)**
   - `Gameplay.tscn` — 테트리스 코어 + HUD
   - `GameOver.tscn` — 게임 오버 화면
   - 일시정지: 씬 아님 — `Gameplay.tscn` 위의 오버레이 노드

2. **단일 진입점 원칙**
   - 모든 씬 전환은 반드시 씬 관리자를 통해서만 수행
   - `get_tree().change_scene_to_file()` 직접 호출 금지 (씬 관리자가 내부적으로 호출)
   - 씬 관리자는 Autoload(`SceneManager`)로 등록

3. **전환 방식**
   - 페이드 아웃 (0.2초) → 씬 교체 → 페이드 인 (0.2초)
   - 전환 중에는 입력 시스템이 `Inactive` 상태 유지
   - 전환 완료 후 입력 시스템을 `Active`로 전환

4. **일시정지 처리**
   - `pause_toggled` 시그널 수신 시: `PauseOverlay` 노드 표시 + `SceneTree.paused = true`
   - `pause_toggled` 재수신 시: `PauseOverlay` 숨김 + `SceneTree.paused = false`
   - 일시정지 중에는 씬 전환 요청 무시 (게임 오버만 예외)

5. **입력 시스템 상태 전환 소유권**
   - 씬 전환 시작 시: `InputSystem.deactivate()` 호출
   - 씬 전환 완료 시: `InputSystem.activate()` 호출
   - 일시정지 진입 시: `InputSystem.pause()` 호출
   - 일시정지 해제 시: `InputSystem.activate()` 호출

### States and Transitions

| 상태 | 설명 | 진입 조건 | 탈출 조건 |
|------|------|----------|----------|
| **Idle** | 현재 씬에서 대기 | 씬 로딩 완료 | 전환 요청 / 일시정지 |
| **Transitioning** | 씬 전환 중 (페이드) | `change_scene()` 호출 | 전환 완료 |
| **Paused** | 일시정지 오버레이 표시 | `pause_toggled` 수신 | `pause_toggled` 재수신 |

**유효한 씬 전환 (MVP):**

| From | To | 트리거 | 전환 효과 |
|------|----|--------|---------|
| (시작) | `Gameplay` | 게임 실행 | 페이드 인 |
| `Gameplay` | `GameOver` | 게임 오버 조건 충족 | 페이드 아웃/인 |
| `GameOver` | `Gameplay` | 재시작 버튼 | 페이드 아웃/인 |

### Interactions with Other Systems

| 시스템 | 방향 | 인터페이스 |
|--------|------|-----------|
| **입력 시스템** | 입력 → 씬 관리자 | `pause_toggled` 시그널 수신 |
| **입력 시스템** | 씬 관리자 → 입력 | `activate()` / `deactivate()` / `pause()` 호출 (상태 전환 소유권) |
| **테트리스 코어** | 코어 → 씬 관리자 | 게임 오버 발생 시 `request_scene_change("GameOver")` 호출 |
| **게임 오버 화면** | GameOver → 씬 관리자 | 재시작 선택 시 `request_scene_change("Gameplay")` 호출 |
| **보상 화면** (Vertical Slice) | 보상 → 씬 관리자 | 다음 스테이지 진행 시 `request_scene_change("Gameplay", {stage_id})` 호출 |
| **메인 메뉴** (Alpha) | 메인메뉴 → 씬 관리자 | 게임 시작 시 `request_scene_change("Gameplay")` 호출 |

## Formulas

씬 관리자의 핵심 계산은 페이드 전환 타이밍뿐이다.

```
# 씬 전환 총 소요 시간
total_transition_time = FADE_OUT_DURATION + FADE_IN_DURATION

# 페이드 진행도 (0.0 ~ 1.0)
fade_progress = elapsed_time / FADE_OUT_DURATION   # 아웃 단계
fade_progress = elapsed_time / FADE_IN_DURATION    # 인 단계
```

| 변수 | 타입 | 기본값 | 안전 범위 | 설명 |
|------|------|--------|---------|------|
| `FADE_OUT_DURATION` | float | 0.2 | 0.1 ~ 0.5 | 페이드 아웃 소요 시간 (초) |
| `FADE_IN_DURATION` | float | 0.2 | 0.1 ~ 0.5 | 페이드 인 소요 시간 (초) |
| `total_transition_time` | float | 0.4 | — | 씬 교체 포함 전체 전환 시간 |

> 씬 교체 자체는 Godot의 `change_scene_to_file()` 호출로 처리. 페이드 아웃 완료 후 호출, 다음 프레임부터 페이드 인 시작.

## Edge Cases

| 시나리오 | 예상 동작 | 근거 |
|---------|----------|------|
| 전환 중 또 다른 전환 요청 | 무시 — `Transitioning` 상태에서는 새 요청을 받지 않음 | 전환 중첩은 씬 깨짐을 유발할 수 있음 |
| 일시정지 중 게임 오버 발생 | 일시정지 즉시 해제 후 게임 오버 씬으로 전환 | 게임 오버는 일시정지보다 우선순위가 높음 |
| 일시정지 중 씬 전환 요청 (재시작 등) | 게임 오버 외 전환 요청은 무시 | 일시정지 상태의 일관성 보장 |
| 존재하지 않는 씬 이름으로 전환 요청 | 오류 로그 출력 후 현재 씬 유지 | 크래시 방지. 개발 중 빠른 오류 발견을 위해 assert 추가 |
| 게임 시작 직후 첫 씬 로딩 | `Gameplay.tscn`을 페이드 아웃 없이 직접 로딩 (첫 실행은 페이드 인만) | 시작 시 검은 화면에서 페이드 인이 자연스러운 진입감을 줌 |
| `FADE_OUT_DURATION = 0` 설정 | 페이드 없이 즉시 씬 교체 | MVP 이전 빠른 테스트 시 유용. 0은 허용 값. |

## Dependencies

| 시스템 | 방향 | 의존 성격 |
|--------|------|---------|
| **입력 시스템** | 씬 관리자가 이 시스템에 의존 | Soft — 입력 시스템 없이도 씬 전환은 가능. 단, 상태 제어를 위해 의존. |
| **테트리스 코어** | 테트리스 코어가 이 시스템에 의존 | Hard — 게임 오버 발생 시 씬 전환 요청을 씬 관리자에 전달 |
| **게임 오버 화면** | 게임 오버 화면이 이 시스템에 의존 | Hard — 재시작 요청을 씬 관리자에 전달 |
| **보상 화면** (Vertical Slice) | 보상 화면이 이 시스템에 의존 | Hard — 다음 스테이지 진행 전환 |
| **메인 메뉴** (Alpha) | 메인 메뉴가 이 시스템에 의존 | Hard — 게임 시작 전환 |

**이 시스템은 다른 시스템에 의존하지 않음** — Foundation Layer 확인 ✅

> 입력 시스템 GDD의 Dependencies 섹션에 "씬 관리자가 이 시스템에 의존 (Soft)"이 명시되어 있어 양방향 일관성 확인됨 ✅

## Tuning Knobs

| 파라미터 | 현재 값 | 안전 범위 | 높이면 | 낮추면 |
|---------|--------|---------|--------|--------|
| `FADE_OUT_DURATION` | 0.2s | 0.0 ~ 0.5 | 전환이 느려져 무게감 증가 | 전환이 빨라지나 갑작스러운 느낌 |
| `FADE_IN_DURATION` | 0.2s | 0.0 ~ 0.5 | 새 씬 등장이 부드럽지만 지연 느낌 | 즉각적인 새 씬 등장 |
| `FADE_COLOR` | `Color.BLACK` | — | — | — |

> `FADE_OUT_DURATION = 0`, `FADE_IN_DURATION = 0`으로 설정하면 테스트 중 씬 전환 효과 없이 즉시 교체 가능. MVP 개발 초기에 유용.

## Visual/Audio Requirements

[미정 — Full Vision 단계에서 결정]

## UI Requirements

[미정 — 게임 HUD GDD 작성 시 일시정지 오버레이 UI 규격 함께 결정]

## Acceptance Criteria

- [ ] `SceneManager` Autoload가 프로젝트에 등록되어 어떤 스크립트에서도 `SceneManager.request_scene_change()` 호출이 가능하다
- [ ] `request_scene_change("Gameplay")` 호출 시 페이드 아웃 → Gameplay 씬 로딩 → 페이드 인 순서로 전환된다
- [ ] `request_scene_change("GameOver")` 호출 시 동일한 전환 흐름으로 GameOver 씬이 로딩된다
- [ ] 씬 전환 중 추가 `request_scene_change()` 호출이 무시된다 (중첩 전환 없음)
- [ ] `pause_toggled` 시그널 수신 시 `PauseOverlay`가 표시되고 `SceneTree.paused = true`가 적용된다
- [ ] `pause_toggled` 재수신 시 `PauseOverlay`가 숨겨지고 `SceneTree.paused = false`가 적용된다
- [ ] 일시정지 상태에서 `request_scene_change()` 호출 시 게임 오버 외에는 무시된다
- [ ] 씬 전환 시작 시 입력 시스템이 비활성화(`Inactive`)되고, 전환 완료 후 활성화(`Active`)된다
- [ ] `FADE_OUT_DURATION`, `FADE_IN_DURATION`, `FADE_COLOR`가 코드에 하드코딩되지 않고 외부 설정에서 읽힌다
- [ ] 존재하지 않는 씬 이름으로 전환 요청 시 오류 로그가 출력되고 현재 씬이 유지된다
- [ ] 씬 전환 총 소요 시간이 `FADE_OUT_DURATION + FADE_IN_DURATION + 1프레임` 이내에 완료된다

## Open Questions

| 질문 | 담당 | 해결 방법 |
|------|------|---------|
| Godot 4.6.1의 `change_scene_to_file()` vs `change_scene_to_packed()` — 씬 경로 하드코딩을 피하는 best practice는? | 구현 시 결정 | WebSearch로 Godot 4.6 공식 문서 확인 후 결정 |
| `SceneTree.paused = true` 적용 시 `PROCESS_MODE_ALWAYS`로 설정해야 하는 노드 목록은? | 구현 시 결정 | 씬 관리자, 입력 시스템은 반드시 ALWAYS 모드 필요 |
| 페이드 효과 구현 방식 — `CanvasLayer` + `ColorRect` 애니메이션 vs Godot Tween? | 구현 시 결정 | 테트리스 코어 구현 시점에 Tween API WebSearch 후 결정 |
| Vertical Slice 이후 메인 메뉴 씬 추가 시 씬 목록을 외부 데이터(Resource)로 관리할 것인가? | Vertical Slice 단계에서 결정 | 씬이 3개 이상이 되는 시점에 리팩토링 여부 검토 |
