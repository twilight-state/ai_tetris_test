# Game Concept: 블록 원정대 (Block Expedition)

*Created: 2026-03-26*
*Status: Draft*

---

## Elevator Pitch

> 테트리스의 클래식한 블록 쌓기 위에 캠페인 구조를 얹은 퍼즐 게임.
> 매 스테이지마다 새로운 미션 조건이 주어지고, 클리어할 때마다 새로운 블록 타입과
> 특수 능력을 해금하며 점점 강해지는 느낌을 즐긴다.

---

## Core Identity

| Aspect | Detail |
| ---- | ---- |
| **Genre** | 퍼즐 / 아케이드 (Tetris-style + Campaign) |
| **Platform** | PC |
| **Target Audience** | 테트리스 경험자, 캐주얼~미드코어 퍼즐 게이머 |
| **Player Count** | 싱글플레이어 |
| **Session Length** | 15~30분 (스테이지 2~3개 클리어) |
| **Monetization** | 없음 (개인 프로젝트) |
| **Estimated Scope** | 소형 (2주) |
| **Comparable Titles** | Tetris Effect, Puyo Puyo Tetris, Tetris 99 |

---

## Core Fantasy

> "내가 이 규칙을 마스터했다" — 처음엔 낯선 미션 조건에 당황하지만,
> 몇 번의 시도 끝에 패턴을 파악하고 깔끔하게 클리어하는 순간의 성취감.
> 새 블록과 스킬이 해금될 때마다 "이걸로 다음 스테이지를 어떻게 공략할까?"라는
> 기대감이 생기는 RPG 성장의 느낌을 테트리스에서 경험한다.

---

## Unique Hook

> "테트리스인데, 매 스테이지마다 규칙이 바뀐다. 그리고 클리어할수록 내 블록이 강해진다."
>
> 일반 테트리스는 하나의 룰로 끝까지 가지만, 블록 원정대는 스테이지마다
> 다른 미션 조건(특정 블록만 사용, 시간 제한, 특정 라인 클리어 목표 등)이
> 주어져 매번 새로운 전략을 요구한다.

---

## Player Experience Analysis (MDA Framework)

### Target Aesthetics (What the player FEELS)

| Aesthetic | Priority | How We Deliver It |
| ---- | ---- | ---- |
| **Sensation** (sensory pleasure) | 3 | 블록 클리어 시 시각/음향 피드백 |
| **Fantasy** (make-believe, role-playing) | N/A | — |
| **Narrative** (drama, story arc) | N/A | — |
| **Challenge** (obstacle course, mastery) | 1 | 스테이지별 미션 조건, 점진적 난이도 |
| **Fellowship** (social connection) | N/A | — |
| **Discovery** (exploration, secrets) | 2 | 새 블록 타입 / 스킬 해금 |
| **Expression** (self-expression, creativity) | 4 | 해금된 스킬 선택 및 전략 수립 |
| **Submission** (relaxation, comfort zone) | N/A | — |

### Key Dynamics (Emergent player behaviors)

- 플레이어는 미션 조건을 먼저 읽고 최적 전략을 미리 계획하게 된다
- 해금된 스킬 중 어느 것을 이 스테이지에 쓸지 고민하게 된다
- "한 스테이지만 더" 심리 — 짧은 스테이지 길이가 반복 플레이를 유도한다

### Core Mechanics (Systems we build)

1. **테트리스 코어** — 블록 낙하, 회전, 라인 클리어
2. **미션 조건 시스템** — 스테이지별 클리어 조건 (시간, 라인 수, 특정 블록 제한 등)
3. **블록 해금 시스템** — 새로운 모양 블록 + 특수 효과 블록 (폭탄, 무지개 등)
4. **스킬 시스템** — 즉발 스킬 (라인 폭파, 시간 슬로우 등) + 패시브 버프 (속도 감소, 점수 배율 등)
5. **캠페인 진행 시스템** — 선형 10스테이지, 클리어 시 다음 스테이지 + 보상 해금

---

## Player Motivation Profile

### Primary Psychological Needs Served

| Need | How This Game Satisfies It | Strength |
| ---- | ---- | ---- |
| **Autonomy** (freedom, meaningful choice) | 해금된 스킬 중 어떤 것을 사용할지 플레이어가 결정 | Supporting |
| **Competence** (mastery, skill growth) | 어려운 미션 조건을 클리어할 때의 성취감, 반복 시도로 실력 향상 | Core |
| **Relatedness** (connection, belonging) | 해당 없음 (싱글플레이어) | Minimal |

### Player Type Appeal (Bartle Taxonomy)

- [x] **Achievers** (goal completion, collection, progression) — 스테이지 클리어, 블록/스킬 수집
- [x] **Explorers** (discovery, understanding systems) — 새 블록/스킬 효과 탐색
- [ ] **Socializers** — 해당 없음
- [ ] **Killers/Competitors** — 해당 없음

### Flow State Design

- **온보딩 커브**: 1~3스테이지는 기본 테트리스와 거의 동일한 조건으로 조작법 학습
- **난이도 스케일링**: 10스테이지에 걸쳐 조건의 복잡도와 속도가 점진적으로 증가
- **피드백 명확성**: 미션 진행도 UI (예: "3/8 라인 클리어"), 스킬 쿨다운 표시
- **실패 후 복귀**: 즉시 재시작 가능, 해금된 블록/스킬은 유지

---

## Core Loop

### Moment-to-Moment (30초)
낙하하는 블록을 회전/이동시켜 쌓고, 라인을 완성해 지운다.
미션 조건에 맞는 블록을 의식적으로 선택하고 배치한다.

### Short-Term (5~10분)
미션 조건을 확인 → 전략 수립 → 플레이 → 성공/실패 → 즉시 재도전.
"이번엔 다른 방식으로 해보자"는 반복 도전 심리가 작동한다.

### Session-Level (15~30분)
2~3개 스테이지를 클리어하며 새 블록/스킬을 해금한다.
자연스러운 종료 시점: 스테이지 클리어 후 보상 화면.
다시 오게 만드는 훅: "다음 스테이지엔 어떤 조건이 나올까? 새 스킬을 써보고 싶다."

### Long-Term Progression
10개 스테이지 선형 클리어가 전체 목표.
클리어할수록 블록 종류 (특수 모양 + 효과 블록) 와 스킬 (즉발 + 패시브) 이 늘어난다.
게임 완료 = 모든 블록/스킬 해금 + 10스테이지 클리어.

### Retention Hooks
- **호기심**: 다음 스테이지의 미션 조건이 공개되지 않음
- **투자**: 해금한 블록/스킬 컬렉션이 쌓여감
- **숙달**: 더 어려운 조건에서 더 빨리 클리어하는 것 자체가 보람

---

## Game Pillars

### Pillar 1: 매 스테이지가 새로운 퍼즐
같은 테트리스 엔진이지만 미션 조건이 달라져 매번 다른 전략이 요구된다.

*Design test*: 새 스테이지가 "그냥 더 빠른 테트리스"인가 vs "다른 방식으로 접근해야 하는 테트리스"인가? → 항상 후자를 선택한다.

### Pillar 2: 성장이 보인다
새 블록과 스킬을 해금할 때 플레이어가 실제로 강해진 느낌을 받아야 한다.

*Design test*: 이 해금 요소가 이후 플레이에 실질적 변화를 주는가? 단순한 숫자 증가라면 재설계한다.

### Pillar 3: 2분 안에 결판
각 스테이지는 짧고 긴장감 있게, 부담 없이 한 판 더 하게 만든다.

*Design test*: 스테이지 평균 클리어 시간이 2분을 넘어가면 조건을 조정한다.

### Anti-Pillars (What This Game Is NOT)

- **NOT 멀티플레이어**: 복잡도만 높이고 2주 범위를 초과함
- **NOT 복잡한 스토리**: 캐릭터 서사보다 플레이 자체가 중심
- **NOT 과도한 랜덤**: 미션 조건은 명확하고 예측 가능해야 하며, 운에 의존하면 안 됨

---

## Inspiration and References

| Reference | What We Take From It | What We Do Differently | Why It Matters |
| ---- | ---- | ---- | ---- |
| Tetris (원작) | 블록 낙하/회전/클리어 코어 메커닉 | 고정 규칙 대신 스테이지별 미션 조건 | 코어 루프의 검증된 재미 |
| StarCraft Campaign | 미션별 다른 조건과 점진적 난이도 | 실시간 전략 대신 퍼즐 기반 | 캠페인 진행의 "다음 스테이지" 긴장감 |
| WoW (캐릭터 성장) | 새로운 능력 해금과 성장 만족감 | 캐릭터 RPG 대신 블록/스킬 해금 | 수집과 성장의 장기 동기부여 |

---

## Target Player Profile

| Attribute | Detail |
| ---- | ---- |
| **Age range** | 20~40대 |
| **Gaming experience** | 캐주얼~미드코어 |
| **Time availability** | 짧은 세션 (15~30분) |
| **Platform preference** | PC |
| **Current games they play** | 테트리스류, 캐주얼 퍼즐, 스타크래프트 |
| **What they're looking for** | 가볍게 시작하지만 점점 빠져드는 퍼즐 |
| **What would turn them away** | 복잡한 튜토리얼, 긴 스테이지, 과도한 난이도 급상승 |

---

## Technical Considerations

| Consideration | Assessment |
| ---- | ---- |
| **Recommended Engine** | **Godot 4** — 무료, 가볍고, 2D 게임에 최적화. GDScript는 Python과 유사해 프로그래밍 경험자에게 배우기 쉬움. 2주 일정에 적합. |
| **Key Technical Challenges** | 미션 조건 시스템 설계 (조건 데이터 구조화), 스킬 시스템 구현 |
| **Art Style** | 2D 픽셀 또는 단순한 2D 벡터 스타일 |
| **Art Pipeline Complexity** | Low — 기하학적 블록 형태로 아트 부담 최소화 |
| **Audio Needs** | Minimal — 효과음 (라인 클리어, 블록 낙하, 스킬 발동) |
| **Networking** | 없음 |
| **Content Volume** | 10스테이지, 10~15종 블록, 6~10개 스킬, 플레이타임 약 1~2시간 |
| **Procedural Systems** | 없음 — 모든 스테이지 조건은 수동 설계 |

---

## Risks and Open Questions

### Design Risks
- 미션 조건이 너무 단순하면 "그냥 테트리스랑 다를게 없다"는 느낌
- 스킬이 너무 강하면 미션 조건이 무의미해짐 (밸런스 필요)

### Technical Risks
- Godot 4 첫 사용 — 기본 기능 학습에 시간 소요 가능
- 스킬 시스템 구현 복잡도가 예상보다 높을 수 있음

### Market Risks
- 해당 없음 (개인 프로젝트)

### Scope Risks
- 10스테이지 × 미션 조건 설계 + 스킬 시스템 구현이 2주를 초과할 수 있음
- 아트 제작 시간 과소평가 가능성

### Open Questions
- 각 스테이지의 미션 조건 상세 목록은? → Phase 3 이후 `/design-system`으로 구체화
- 스킬 선택 UI는 게임 중 언제 나타나는가? → 프로토타입으로 검증

---

## MVP Definition

**Core hypothesis**: "스테이지별 미션 조건이 테트리스 코어 루프에 충분한 다양성과 재미를 더한다"

**Required for MVP**:
1. 기본 테트리스 코어 (블록 낙하, 회전, 라인 클리어)
2. 미션 조건 시스템 (최소 3가지 조건 타입)
3. 3~5스테이지 선형 진행 + 스테이지 클리어 보상 화면

**Explicitly NOT in MVP** (defer to later):
- 특수 효과 블록 (폭탄, 무지개)
- 즉발 스킬 / 패시브 버프 시스템
- 10스테이지 전체 콘텐츠
- 정식 아트 / 사운드

### Scope Tiers

| Tier | Content | Features | Timeline |
| ---- | ---- | ---- | ---- |
| **MVP** | 3~5스테이지 | 테트리스 코어 + 미션 조건 | 1주차 |
| **Vertical Slice** | 5~7스테이지 | + 새 블록 타입 해금 | 1.5주차 |
| **Alpha** | 10스테이지 | + 스킬 시스템 (즉발 + 패시브) | 2주차 |
| **Full Vision** | 10스테이지 (폴리시) | 특수 효과 블록 + 사운드 + UI 완성 | 2주차 후반 |

---

## Next Steps

- [ ] `/setup-engine godot 4` — Godot 4 엔진 설정 및 버전 레퍼런스 구성
- [ ] `/design-review design/gdd/game-concept.md` — 컨셉 문서 검토
- [ ] `/map-systems` — 시스템 분해 및 의존성 매핑
- [ ] `/design-system` — 미션 조건 시스템 GDD 작성
- [ ] `/prototype tetris-core` — 테트리스 코어 루프 프로토타입
- [ ] `/sprint-plan new` — 첫 번째 스프린트 계획
