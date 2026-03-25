# Godot — Version Reference

*Last verified: 2026-03-26*

| Field | Value |
|-------|-------|
| **Engine Version** | Godot 4.6.1-stable |
| **Project Pinned** | 2026-03-26 |
| **LLM Knowledge Cutoff** | August 2025 (Godot ~4.3 coverage) |
| **Risk Level** | HIGH — 4.4, 4.5, 4.6 변경사항은 학습 데이터 이후 |

## 지식 격차 요약

이 프로젝트는 Godot 4.6.1을 사용합니다. LLM의 학습 데이터는 대략 Godot 4.3까지
커버합니다. 에이전트가 코드를 제안할 때는 반드시 아래 레퍼런스 문서를 먼저 확인해야 합니다.

## 레퍼런스 문서

| 파일 | 내용 |
|------|------|
| `breaking-changes.md` | 4.3→4.6 버전별 주요 변경사항 |
| `current-best-practices.md` | Godot 4.6 기준 모범 사례 |

## 공식 문서 링크

- 마이그레이션 가이드 (4.3→4.4): https://docs.godotengine.org/en/4.4/tutorials/migrating/upgrading_to_godot_4.4.html
- 마이그레이션 가이드 (4.4→4.5): https://docs.godotengine.org/en/4.5/tutorials/migrating/upgrading_to_godot_4.5.html
- 마이그레이션 가이드 (4.5→4.6): https://docs.godotengine.org/en/stable/tutorials/migrating/upgrading_to_godot_4.6.html
- Godot 4.6 릴리즈 노트: https://godotengine.org/releases/4.6/

## 에이전트 지시사항

코드를 제안하기 전에:
1. 이 파일을 읽어 버전 확인
2. `breaking-changes.md`에서 관련 API 변경 여부 확인
3. `current-best-practices.md`에서 현재 권장 패턴 확인
4. 불확실한 API는 WebSearch로 공식 문서 검증
