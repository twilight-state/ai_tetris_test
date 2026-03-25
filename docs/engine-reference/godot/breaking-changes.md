# Godot Breaking Changes: 4.3 → 4.6

*Last verified: 2026-03-26*

이 문서는 LLM 학습 데이터(~4.3) 이후의 주요 변경사항을 정리합니다.
코드 제안 전 반드시 확인하세요.

---

## Godot 4.3 → 4.4

공식 문서: https://docs.godotengine.org/en/4.4/tutorials/migrating/upgrading_to_godot_4.4.html

### GDScript 변경사항
- 대부분의 4.3 GDScript 코드는 4.4에서 그대로 동작
- 마이그레이션 가이드에서 특정 deprecated API 확인 권장

### 기타
- 4.3 → 4.4 마이그레이션은 대부분의 프로젝트에서 안전
- 프로젝트 열기 시 자동 변환 제안

---

## Godot 4.4 → 4.5

공식 문서: https://docs.godotengine.org/en/4.5/tutorials/migrating/upgrading_to_godot_4.5.html

### 3D 물리 (2D 게임은 영향 없음)
- 3D fixed-timestep interpolation 완전 재설계
- SceneTree 내부에서 처리되도록 변경 (기존 API는 유지)

### UI/Control 개선
- `FoldableContainer` 신규 노드 추가
- Control 노드의 마우스/포커스 동작을 재귀적으로 변경 가능
- Label 텍스트 효과 레이어 스택 지원

### 에디터
- 다중 노드 선택 후 공통 속성 일괄 편집 가능

### 2D 게임 관련 영향
- **낮음** — 대부분의 변경사항은 3D 및 에디터 관련

---

## Godot 4.5 → 4.6

공식 문서: https://docs.godotengine.org/en/stable/tutorials/migrating/upgrading_to_godot_4.6.html

### ⚠️ 물리 엔진 기본값 변경 (중요)
- **신규 프로젝트**의 3D 기본 물리 엔진이 **Jolt Physics**로 변경
- 기존 프로젝트는 영향 없음 (설정 유지)
- 2D 게임은 해당 없음

### ⚠️ 렌더링 — Glow 기본값 변경
- Glow 기본 블렌드 모드가 `Soft Light` → `Screen`으로 변경
- Screen 모드가 더 밝게 보임
- 기존 프로젝트에서 Glow 사용 중이라면 조정 필요
- **2D 퍼즐 게임에서는 대부분 영향 없음**

### ⚠️ GLSL 셰이더 — View Matrix 변경
- 셰이더 코드에서 view matrix 관련 변경사항 있음
- 커스텀 셰이더 사용 시 공식 마이그레이션 가이드 확인 필수

### 노드 관리 개선
- 노드에 내부 고유 ID 추가 — 씬 재구성 시 참조 유지
- 노드 이름 변경/이동 시 참조가 끊기지 않음

### 에디터
- `EditorSettings`를 통한 커스텀 키보드 단축키 등록 가능
- 디버거에 **Step Out** 버튼 추가 (Step Over, Step Into와 함께)

### GDScript
- 코루틴에 `await` 없이 호출 시 경고 옵트인 추가
- Tracy 프로파일러 네이티브 지원

### Android (모바일 게임 해당)
- 빌드 디렉토리 구조 변경: `build/src/` → `build/src/main/java/`

---

## 이 프로젝트(블록 원정대)에 대한 영향 요약

| 변경사항 | 영향 |
|---------|------|
| Jolt Physics 기본값 | ❌ 없음 (2D 게임) |
| Glow 기본값 변경 | ⚠️ 낮음 (Glow 미사용 시 무관) |
| GLSL 셰이더 변경 | ⚠️ 커스텀 셰이더 사용 시 확인 |
| 노드 ID 추가 | ✅ 유리함 |
| Step Out 디버거 | ✅ 개발 편의성 향상 |
| GDScript await 경고 | ✅ 코드 품질 향상 |
