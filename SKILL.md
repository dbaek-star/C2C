---
name: c2c-collab
description: |
  Claude-A(주 에이전트)와 Claude-B(서브에이전트) 간의 구조화된 라운드 기반 협업.
  독립된 컨텍스트에서 교차 검증하여 더 높은 품질의 결과물을 생성한다.
  Triggers: "C2C 협업", "서브에이전트와 협업", "클로드 협업",
  "Claude 협업", "C2C collab", "collaborate with subagent",
  "AI 자체 협업", "두 클로드 협업", "서브에이전트와 논의",
  "서브에이전트한테 물어봐", "AI끼리 토론", "두 AI 의견 비교".
---

# C2C Collaboration (Round-based)

Claude-B 호출은 Agent tool (subagent)로 수행. 파라미터·응답 처리·장애 처리 등 상세는 [references/subagent-common.md](references/subagent-common.md) 참조.

**경로 규칙:**
- `{CWD}` = Claude의 현재 작업 디렉토리. 스킬 호출 시점의 CWD를 기준으로 한다.
- 모든 산출물은 반드시 `{CWD}/.c2c/collab/` 하위에 저장할 것. **절대로 스킬 정의 디렉토리(`~/.claude/skills/`)나 다른 경로에 저장하지 말 것.**

**스킬 고유 규칙:**
- 고정 프롬프트 사용 금지. 매 서브에이전트 호출 시 주제·라운드·모드·이전 피드백을 고려하여 prompt를 자율 생성할 것
- 첫 라운드에서 WebSearch로 최신 정보를 조사한 뒤 초안 작성할 것

## 실행 방식

스킬 호출 시 반드시 다음 순서로 실행할 것:

```
1. AskUserQuestion으로 협업 모드 + 서브에이전트 모델 선택 (동시)
1.5. (최초 1회) .gemini/context.md 존재 시 → .c2c/context.md로 복사
2. 출력 폴더 생성: {CWD}/.c2c/collab/{YYYYMMDD_HHMMSS}_{주제요약}/
3. 모드별 협업 수행 (references/modes.md 참조)
4. collab_summary.md 생성
5. context.md 업데이트
6. 사용자에게 결과 전달
```

### 초기 설정 (AskUserQuestion - 2개 질문 동시)

```
questions:
  - question: "협업 모드를 선택해주세요."
    header: "협업 모드"
    options:
      - label: "2 Round (Recommended)"
        description: "초안→검토→수정→재검토→최종. 균형 잡힌 기본 모드"
      - label: "1 Round"
        description: "초안→검토→최종. 빠른 협업"
      - label: "Adaptive Round"
        description: "양쪽 합의 시까지 반복 (최대 5라운드)"
      - label: "Devil's Advocate"
        description: "비판적 토론, 한쪽 패배 시까지 (최대 10라운드, 연장 가능)"
  - question: "서브에이전트(Claude-B) 모델을 선택해주세요."
    header: "서브에이전트 모델"
    options:
      - label: "Claude Sonnet (Recommended)"
        description: "빠르고 균형잡힌 성능. 대부분의 협업에 적합"
      - label: "Claude Opus"
        description: "최고 성능. 복잡한 추론·분석에 적합"
      - label: "Claude Haiku"
        description: "경량 고속. 단순 작업에 적합"
```

모델 선택값과 Agent `model` 파라미터 매핑:

| 사용자 선택 | `model` 값 |
|------------|-----------|
| Claude Sonnet | `"sonnet"` |
| Claude Opus | `"opus"` |
| Claude Haiku | `"haiku"` |
| (Other 입력) | 입력값 그대로 |

## 협업 모드 개요

4가지 모드를 지원. 각 모드의 상세 흐름, 파일명 규칙, 종료 조건, 수행 절차는 [references/modes.md](references/modes.md) 참조.

| 모드 | 흐름 | 서브에이전트 호출 |
|------|------|-----------------|
| [1] 1 Round | 초안→검토→판단→최종 | 1회 |
| [2] 2 Round (기본) | 초안→검토→수정→재검토→최종판단→최종 | 2회 |
| [3] Adaptive | [2] 반복, 합의 시 종료 | 1~5회 |
| [4] Devil's Advocate | 주장→반박 무한 반복, 패배 선언 시 종료 | 최대 10회 (연장 가능) |

## 서브에이전트 호출 패턴

매 서브에이전트 호출 시 다음 형식으로 Agent tool을 사용한다:

```
Agent({
  description: "Round {N} - {역할 설명}",
  subagent_type: "general-purpose",
  model: "{사용자 선택 모델}",
  prompt: "{동적 생성된 프롬프트}"
})
```

상세 파라미터 및 prompt 구조는 [references/subagent-common.md](references/subagent-common.md) 참조.

### 장애 처리

1. Agent 호출 실패 → 동일 파라미터로 **1회 재시도**
2. 재시도 실패 → Claude-A 단독 수행, `collab_summary.md`에 기록
3. 단독 수행 시에도 동일한 산출물 구조 유지

## 프롬프트 생성 규칙

매 서브에이전트 호출 시 Claude-A가 prompt를 자율 생성할 것.

**고려 요소**: 주제, 현재 라운드 번호, 선택된 모드, 이전 피드백 내용

**웹검색 지시 규칙** (prompt 생성 시 주제에 따라 적용):

| 주제 유형 | 강도 | prompt에 추가할 문구 |
|-----------|------|---------------------|
| 팩트체크/최신정보 필요 | 강제 | "반드시 WebSearch를 수행하여 정보의 정확성을 교차 검증하라. 검색 결과가 불충분할 경우, 그 사실을 명시하고 내부 지식 기반으로 답변하되 불확실성을 표시하라" |
| 코드리뷰/로직분석/번역 | 생략 | 별도 지시 불필요 |
| 판단 불확실 | 권장 | "가능하면 WebSearch로 핵심 주장의 사실 여부를 확인하라" |

주제 유형 판별:
- **강제**: 최신 기술 트렌드, 버전 정보, 통계/날짜/인용 검증, 시장 분석, API 문서 확인
- **생략**: 코드 리뷰, 버그 분석, 아키텍처 토론, 번역/교정, 이전 라운드 피드백 검토
- **권장**: 위 두 카테고리에 명확히 속하지 않거나 사실 정보와 주관적 분석이 혼재된 경우

**모드별 톤**: [references/modes.md](references/modes.md)의 "프롬프트 톤" 섹션 참조.

## 독립성 보장

서브에이전트(Claude-B)는 주 에이전트(Claude-A)의 사고 과정에 접근할 수 없고, 결과물(초안)만 받아 독립된 컨텍스트에서 평가한다. 작성자와 검토자의 역할 차이가 자연스러운 관점 다양성을 만든다.

**페르소나 템플릿:**

| 모드 | 서브에이전트 페르소나 |
|------|---------------------|
| 1 Round / 2 Round / Adaptive | "독립적 리뷰어. 동의/문제점을 솔직하게 판단. 억지 비판 금지, 동조 압력 없이 평가" |
| Devil's Advocate | "극단적 반론자. 논리적 허점·근거 부족 적극 공격. 반박 불가 시만 패배 인정" |

- 리뷰 모드: 정량적 제약 없음. 실제로 문제가 있을 때만 지적하고, 잘된 부분은 인정
- Devil's Advocate: 모드 특성상 공격적 반론 유지

## 컨텍스트 관리

다중 라운드에서의 Context Bloat 방지:

| 라운드 | 컨텍스트 전달 방식 |
|--------|-------------------|
| Round 1 | 초안 전문을 prompt에 포함 |
| Round 2 | 초안 전문 + Round 1 리뷰 전문을 prompt에 포함 |
| Round 3+ | Claude-A가 작성한 **요약만** prompt에 포함 + 이전 라운드 파일 경로를 제공하여 Read tool로 참조 유도 |

Round 3 이상의 prompt 요약 구조:
```
## 이전 라운드 요약
- 핵심 쟁점: ...
- 합의된 사항: ...
- 미합의 사항: ...

참고 파일 (필요 시 Read tool로 직접 확인):
- {이전 라운드 파일 경로 목록}
```

## 산출물

`{CWD}/.c2c/collab/{YYYYMMDD_HHMMSS}_{주제요약}/`에 저장.

**파일명 규칙**: `roundN_M_주체_유형.md` (N=라운드, M=순번, 주체=claude/claude-b, 유형=draft/review/decision 등)

**고정 파일**: `collab_final.md` (최종본), `collab_summary.md` (요약)

## 요약 생성 (collab_summary.md)

포함 항목: 사용 모델(서브에이전트), 모드, 총 라운드 수, 총 서브에이전트 호출 횟수, 원래 요청 요약, 라운드별 주요 결정 (반영/미반영 항목과 이유), 작업 결과물 요약, 산출물 위치.
- [3] Adaptive: 합의/강제종료 여부 추가
- [4] Devil's Advocate: 승패 결과, 핵심 논점 추가

## context.md 업데이트

`.c2c/context.md`에 추가:
```
## 최근 협업 (collab)
- {날짜}: {주제} ({모드}, {라운드}R) → 결과: {요약}
```

## 사용자에게 결과 전달

- `collab_summary.md` 핵심 내용 전달
- Agent 호출 실패/단독 수행이 있었으면 명확히 전달
- 상세 내용은 `.c2c/collab/` 폴더 참조 안내

## 주의사항

- Agent 호출 실패 시 반드시 사용자에게 알림
- [4] Devil's Advocate: 종료 조건·금지사항은 [references/modes.md](references/modes.md) 참조
