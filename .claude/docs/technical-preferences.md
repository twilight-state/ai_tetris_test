# Technical Preferences

<!-- Populated by /setup-engine. Updated as the user makes decisions throughout development. -->
<!-- All agents reference this file for project-specific standards and conventions. -->

## Engine & Language

- **Engine**: Godot 4.6.1
- **Language**: GDScript (primary), C++ via GDExtension (성능 크리티컬 시스템에 한해 사용)
- **Rendering**: Godot Rendering Server (2D — CanvasItem / Viewport)
- **Physics**: Jolt Physics (Godot 4.6 신규 프로젝트 기본값)

## Naming Conventions

- **Classes**: PascalCase (예: `BlockController`, `MissionManager`)
- **Variables/Functions**: snake_case (예: `move_speed`, `clear_line()`)
- **Signals**: snake_case 과거형 (예: `line_cleared`, `stage_completed`)
- **Files**: snake_case, 클래스명과 일치 (예: `block_controller.gd`)
- **Scenes**: PascalCase, 루트 노드명과 일치 (예: `BlockController.tscn`)
- **Constants**: UPPER_SNAKE_CASE (예: `MAX_FALL_SPEED`, `BOARD_WIDTH`)

## Performance Budgets

- **Target Framerate**: 60fps
- **Frame Budget**: 16.6ms
- **Draw Calls**: [TO BE CONFIGURED — 2D 퍼즐 게임이므로 낮을 것으로 예상]
- **Memory Ceiling**: [TO BE CONFIGURED]

## Testing

- **Framework**: GUT (Godot Unit Testing) — `addons/gut/`
- **Minimum Coverage**: [TO BE CONFIGURED]
- **Required Tests**: 미션 조건 판정 로직, 블록 충돌/클리어 시스템, 스킬 효과 계산

## Forbidden Patterns

<!-- Add patterns that should never appear in this project's codebase -->
- 게임플레이 수치 하드코딩 금지 — 모든 밸런스 값은 외부 데이터(Resource/JSON)로 관리
- 싱글턴 남용 금지 — 의존성 주입(Dependency Injection) 우선 사용
- `get_node()` 문자열 경로 직접 사용 금지 — `@onready var` 또는 typed 노드 참조 사용

## Allowed Libraries / Addons

- **GUT** — 유닛 테스트 프레임워크 (승인됨)

## Architecture Decisions Log

<!-- Quick reference linking to full ADRs in docs/architecture/ -->
- [No ADRs yet — use /architecture-decision to create one]
