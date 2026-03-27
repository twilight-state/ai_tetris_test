# 게임 HUD (Game HUD)

> **Status**: In Design
> **Author**: Luna/Solar Eclipse + Claude
> **Last Updated**: 2026-03-28
> **Implements Pillar**: 피드백 명확성 (플레이어가 미션 진행도와 게임 상태를 한눈에 파악)

## Overview

게임 HUD는 게임플레이 화면에서 플레이어가 의사결정에 필요한 정보를 실시간으로 표시하는 시스템이다. 테트리스 코어로부터 활성 피스·Ghost Piece·Hold 슬롯·Next 프리뷰를 수신해 렌더링하고, 미션 조건 시스템으로부터 진행도·남은 시간을 수신해 표시한다. HUD는 직접 게임 상태를 계산하지 않으며 — 모든 데이터는 시그널로 수신한다. 이 시스템이 없으면 플레이어는 다음 블록이 무엇인지, 미션 달성에 얼마나 근접했는지 알 수 없다.

## Player Fantasy

HUD는 플레이어가 의식하지 않아야 하지만, 없으면 즉시 느껴지는 시스템이다. "다음 블록이 I피스다 — 여기에 끼워넣으면 테트리스가 된다"는 전략적 판단이 0.5초 안에 이루어질 수 있는 것은 Next 프리뷰 덕분이다. 미션 진행도 바가 서서히 차오를 때의 기대감, 남은 시간이 5초 이하로 떨어지면서 타이머가 빨갛게 변할 때의 극도의 집중 — 이 감각적 정보들이 플레이어를 Flow 상태로 유지시킨다. 게임 컨셉의 "피드백 명확성: 미션 진행도 UI, 스킬 쿨다운 표시"가 이 시스템의 존재 이유다.

## Detailed Design

### Core Rules

[To be designed]

### States and Transitions

[To be designed]

### Interactions with Other Systems

[To be designed]

## Formulas

[To be designed]

## Edge Cases

[To be designed]

## Dependencies

[To be designed]

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
