---
name: c2c-collab
description: |
  Claude-A(주 에이전트)와 Claude-B(서브에이전트) 간의 구조화된 라운드 기반 협업.
  독립된 컨텍스트에서 교차 검증하여 더 높은 품질의 결과물을 생성한다.
  Use when: 기술 문서 작성, 시스템 아키텍처 설계, 기술적 의사결정 토론,
  PRD 작성, 코드 리뷰 협업, 복잡한 분석 등 두 관점의 교차 검증이 필요한 작업.
  Triggers: "C2C 협업", "서브에이전트와 협업", "클로드 협업",
  "Claude 협업", "C2C collab", "collaborate with subagent",
  "AI 자체 협업", "두 클로드 협업", "서브에이전트와 논의",
  "서브에이전트한테 물어봐", "AI끼리 토론", "두 AI 의견 비교".
---

# C2C Collaboration (Round-based)

Claude-B는 Agent tool (subagent, `subagent_type: "general-purpose"`)로 호출. 상세 호출 규칙은 [references/subagent-common.md](references/subagent-common.md) 참조.

**경로 규칙:**
- 모든 산출물은 반드시 `{CWD}/.c2c/collab/` 하위에 저장할 것. **절대로 스킬 정의 디렉토리에 저장하지 말 것.**

## 실행 방식

```
1. AskUserQuestion으로 협업 모드 + 서브에이전트 모델 선택 (동시)
1.5. (최초 1회) .gemini/context.md 존재 시 → .c2c/context.md로 복사
2. 출력 폴더 생성: {CWD}/.c2c/collab/{YYYYMMDD_HHMMSS}_{주제요약}/
3. 모드별 협업 수행 (references/modes.md 참조)
4. collab_summary.md 생성
5. .c2c/context.md 업데이트
6. 사용자에게 결과 전달
```

### 초기 설정 (AskUserQuestion - 2개 질문 동시)

질문 1 — 협업 모드:
- "2 Round (Recommended)": 초안→검토→수정→재검토→최종
- "1 Round": 초안→검토→최종
- "Adaptive Round": 양쪽 합의 시까지 반복 (최대 5라운드)
- "Devil's Advocate": 비판적 토론 (최대 10라운드, 연장 가능)

질문 2 — 서브에이전트 모델:
- "Claude Sonnet (Recommended)": 빠르고 균형잡힌 성능
- "Claude Opus": 최고 성능, 복잡한 추론
- "Claude Haiku": 경량 고속

## 협업 모드

4가지 모드 지원. 상세 흐름·파일명·종료 조건은 [references/modes.md](references/modes.md) 참조.

| 모드 | 흐름 | 서브에이전트 호출 |
|------|------|-----------------|
| 1 Round | 초안→검토→판단→최종 | 1회 |
| 2 Round (기본) | 초안→검토→수정→재검토→최종 | 2회 |
| Adaptive | 2 Round 반복, 합의 시 종료 | 1~5회 |
| Devil's Advocate | 주장→반박 반복, 패배 선언 시 종료 | 최대 10회 |

## 프롬프트 생성 규칙

매 서브에이전트 호출 시 prompt를 자율 생성할 것. 고정 프롬프트 사용 금지.

**고려 요소**: 주제, 라운드 번호, 모드, 이전 피드백

**웹검색 지시** (주제 유형에 따라):

| 주제 유형 | 지시 |
|-----------|------|
| 팩트체크/최신정보 (기술 트렌드, 버전, 통계, API 문서) | 강제: "반드시 WebSearch로 정보를 교차 검증하라" |
| 코드리뷰/로직분석/번역 | 생략 |
| 판단 불확실 | 권장: "가능하면 WebSearch로 핵심 주장을 확인하라" |

## 독립성 보장

서브에이전트는 주 에이전트의 사고 과정에 접근할 수 없고, 결과물만 받아 독립 평가한다.

**페르소나:**

| 모드 | 서브에이전트 페르소나 |
|------|---------------------|
| 1R / 2R / Adaptive | "독립적 리뷰어. 솔직하게 판단. 억지 비판 금지, 동조 압력 없이 평가" |
| Devil's Advocate | "극단적 반론자. 논리적 허점·근거 부족 적극 공격. 반박 불가 시만 패배 인정" |

## 산출물

`{CWD}/.c2c/collab/{YYYYMMDD_HHMMSS}_{주제요약}/`에 저장.

**파일명**: `roundN_M_주체_유형.md` (주체=claude/claude-b, 유형=draft/review/decision 등)
**고정 파일**: `collab_final.md`, `collab_summary.md`

### collab_summary.md 포함 항목

사용 모델, 모드, 총 라운드 수, 서브에이전트 호출 횟수, 원래 요청 요약, 라운드별 주요 결정(반영/미반영과 이유), 결과물 요약, 산출물 위치.
- Adaptive: 합의/강제종료 여부
- Devil's Advocate: 승패 결과, 핵심 논점

### context.md

`.c2c/context.md`에 `- {날짜}: {주제} ({모드}, {라운드}R) → 결과: {요약}` 형식으로 추가.

## 결과 전달

- `collab_summary.md` 핵심 내용 전달
- Agent 호출 실패 시 명확히 알림
- 상세 내용은 `.c2c/collab/` 폴더 참조 안내
