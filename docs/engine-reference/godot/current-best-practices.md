# Godot 4.6 — 현재 모범 사례

*Last verified: 2026-03-26*

공식 문서: https://docs.godotengine.org/en/4.6/tutorials/best_practices/

---

## GDScript 코드 스타일

### 정적 타이핑 (강력 권장)
```gdscript
# 권장
var speed: float = 5.0
func move(direction: Vector2) -> void:
    pass

# 지양
var speed = 5.0
func move(direction):
    pass
```

### @onready 변수
```gdscript
# 권장 — 타입 명시
@onready var board: GridContainer = $Board
@onready var score_label: Label = $HUD/ScoreLabel

# 지양 — 문자열 경로 직접 사용
var board = get_node("Board")
```

### 시그널 (Signal)
```gdscript
# 선언
signal line_cleared(count: int)
signal stage_completed(stage_id: int)

# 발신
line_cleared.emit(4)

# 연결 (코드)
other_node.line_cleared.connect(_on_line_cleared)

# 연결 (에디터) — 가능하면 에디터에서 연결 권장
```

### 코루틴과 await
```gdscript
# await 없이 코루틴 호출 시 4.6부터 경고 발생
# 반드시 await 사용
await get_tree().create_timer(1.0).timeout

# 잘못된 예 (4.6에서 경고)
get_tree().create_timer(1.0).timeout  # await 없음
```

---

## 씬(Scene) 구조 설계

### 씬 분리 원칙
- 각 게임 요소를 독립적인 씬으로 분리
- 씬은 재사용 가능하게 설계 (부모에 의존하지 않음)
- 루트 노드가 씬의 책임을 명확히 표현해야 함

### 블록 원정대 권장 씬 구조
```
Main.tscn
├── GameBoard.tscn      # 게임 보드 로직
├── BlockSpawner.tscn   # 블록 생성/관리
├── MissionPanel.tscn   # 미션 조건 UI
├── SkillBar.tscn       # 스킬 UI
└── HUD.tscn            # 점수/시간 표시
```

---

## 노드 통신 패턴

### 아래로 호출, 위로 시그널 (핵심 원칙)
```gdscript
# 부모 → 자식: 직접 메서드 호출 OK
$BlockSpawner.spawn_block(BlockType.L_SHAPE)

# 자식 → 부모: 시그널 사용 (직접 참조 금지)
signal line_cleared(count: int)  # 자식에서 선언
# 부모가 연결해서 처리
```

### 노드 참조
```gdscript
# 권장 — @onready + 타입
@onready var spawner: BlockSpawner = $BlockSpawner

# 지양 — 런타임 get_node
var spawner = get_node("/root/Main/BlockSpawner")
```

---

## Resource 활용 (데이터 외부화)

### 게임 데이터를 Resource로 관리
```gdscript
# StageData.gd
class_name StageData
extends Resource

@export var stage_id: int
@export var mission_type: MissionType
@export var target_value: int
@export var time_limit: float
@export var unlocks_block: BlockType
@export var unlocks_skill: SkillType
```

```gdscript
# 사용
var stage: StageData = preload("res://assets/data/stages/stage_01.tres")
```

- 모든 스테이지 데이터, 블록 데이터, 스킬 데이터는 `.tres` Resource 파일로 관리
- 수치를 코드에 하드코딩하지 않음 (이 프로젝트의 Forbidden Pattern)

---

## 물리 (2D)

### CharacterBody2D vs Area2D vs StaticBody2D
- 테트리스 블록: `Area2D` 또는 단순 위치 계산 (물리 엔진 불필요)
- 테트리스는 그리드 기반이므로 물리 엔진보다 **직접 좌표 계산** 권장

---

## 4.6 신기능 활용 팁

### 노드 고유 ID
- 4.6부터 노드에 내부 고유 ID가 생겨 씬 재구성 시 참조 유지
- 에디터에서 노드 이름을 바꿔도 참조가 유지됨

### Step Out 디버거
- 디버그 중 함수 내부로 들어갔다가 빠져나올 때 Step Out 사용
- 중단점 없이 현재 함수 완료 후 호출 지점으로 복귀

---

## 테스트 (GUT)

```gdscript
# test_board.gd
extends GutTest

func test_line_clear_removes_row():
    var board = preload("res://src/gameplay/GameBoard.tscn").instantiate()
    add_child(board)
    board.fill_row(0)
    board.check_and_clear_lines()
    assert_eq(board.get_row_fill(0), 0, "Row should be cleared")
    board.queue_free()
```

- 테스트 파일은 `tests/unit/` 에 위치
- 파일명 `test_*.gd` 패턴 사용
