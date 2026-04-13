<div align="center">

# C2C Collab

### Claude-A × Claude-B — AI 간 협업 지능 시스템

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-7C3AED?style=for-the-badge&logo=anthropic&logoColor=white)](https://docs.anthropic.com/en/docs/claude-code)
[![License: MIT](https://img.shields.io/badge/License-MIT-F59E0B?style=for-the-badge)](LICENSE)

<br/>

**두 개의 Claude 인스턴스. 하나의 통합된 결과물.**
C2C Collab은 Claude-A(오케스트레이터)와 Claude-B(서브에이전트) 사이의 구조화된 라운드 기반 협업을 오케스트레이션합니다. 독립적인 평가, 교차 검증, 반복적 개선을 통해 더 높은 품질의 결과물을 만들어냅니다.

<br/>

[English README](README.md)

---

<img src="https://img.shields.io/badge/초안-Claude--A-7C3AED?style=flat-square" alt="Claude-A"> →
<img src="https://img.shields.io/badge/검토-Claude--B-2563EB?style=flat-square" alt="Claude-B"> →
<img src="https://img.shields.io/badge/판단-Claude--A-7C3AED?style=flat-square" alt="Claude-A"> →
<img src="https://img.shields.io/badge/최종-합의-10B981?style=flat-square" alt="Final">

</div>

<br/>

## 목차

- [왜 C2C Collab인가?](#-왜-c2c-collab인가)
- [작동 방식](#-작동-방식)
- [협업 모드](#-협업-모드)
- [사전 요구사항](#-사전-요구사항)
- [설치 방법](#-설치-방법)
- [사용 방법](#-사용-방법)
- [모델 지원](#-모델-지원)
- [산출물 구조](#-산출물-구조)
- [아키텍처](#-아키텍처)
- [사용 예시](#-사용-예시)
- [기여하기](#-기여하기)

---

## 왜 C2C Collab인가?

> 하나의 AI가 초안을 작성할 수 있다. 두 개의 AI 인스턴스는 **검증하고, 도전하고, 개선**할 수 있다.

| 문제점 | 해결책 |
|:-------|:-------|
| 단일 컨텍스트의 맹점 | 서브에이전트가 완전히 독립된 컨텍스트에서 평가 |
| 검증 없는 가정 | 구조화된 검토 라운드로 솔직한 평가 유도 |
| 환각(Hallucination) 위험 | 웹 검색 통합 + 이중 검증 |
| 작성자 편향 | 역할 분리(작성자/검토자)로 자연스러운 관점 다양성 확보 |

**C2C Collab**은 Claude Code의 내장 Agent tool을 활용하여 독립적인 서브에이전트를 생성합니다 — 외부 CLI도, 스크립트도, Claude Code 외의 추가 의존성도 없습니다.

---

## 작동 방식

```
┌─────────────────────────────────────────────────────────┐
│                      C2C COLLAB                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│   │ CLAUDE-A  │───▶│ CLAUDE-B │───▶│ CLAUDE-A  │        │
│   │   초안    │    │   검토   │    │   판단    │        │
│   │ +웹검색   │    │(서브에이전트)│  │ 수용/     │        │
│   └──────────┘    └──────────┘    │ 반박      │        │
│                                    └─────┬────┘         │
│                                          │              │
│                              ┌───────────▼───────────┐  │
│                              │     최종 산출물       │  │
│                              │  collab_final.md      │  │
│                              │  collab_summary.md    │  │
│                              └───────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

1. **Claude-A가 조사** — 웹 검색으로 주제를 조사한 후 초안을 작성합니다
2. **Claude-B가 검토** — 독립된 컨텍스트에서 초안을 검토합니다 (자체 웹 검색 포함)
3. **Claude-A가 평가** — Claude-B의 피드백을 수용, 반박, 또는 부분 채택합니다
4. **(선택)** 합의 또는 종료 조건까지 추가 라운드를 진행합니다
5. **최종 산출물** — 종합적인 협업 요약과 함께 최종본이 생성됩니다

---

## 협업 모드

<table>
<tr>
<td width="25%" align="center">

### 1 Round
**빠른 협업**

</td>
<td width="25%" align="center">

### 2 Round
**추천 (기본값)**

</td>
<td width="25%" align="center">

### Adaptive
**합의까지 반복**

</td>
<td width="25%" align="center">

### Devil's Advocate
**토론 모드**

</td>
</tr>
<tr>
<td>

```
Claude-A 초안
    ↓
Claude-B 검토
    ↓
Claude-A 판단
    ↓
  최종본
```

</td>
<td>

```
Claude-A 초안
    ↓
Claude-B 검토
    ↓
Claude-A 수정
    ↓
Claude-B 재검토
    ↓
Claude-A 최종 판단
    ↓
  최종본
```

</td>
<td>

```
Claude-A 초안
    ↓
┌─── 반복 ────┐
│ Claude-B 검토│
│     ↓        │
│ Claude-A 판단│
└─── × N ─────┘
    ↓
  최종본
```

</td>
<td>

```
Claude-A 주장
    ↓
Claude-B 반박
    ↓
Claude-A 재반박
    ↓
    ...
    ↓
 패배 선언!
```

</td>
</tr>
<tr>
<td>

서브에이전트 호출: **1회**
적합: 빠른 검토

</td>
<td>

서브에이전트 호출: **2회**
적합: 대부분의 작업

</td>
<td>

서브에이전트 호출: **1~5회**
적합: 복잡한 주제

</td>
<td>

서브에이전트 호출: **최대 10회**
적합: 논쟁적 주제

</td>
</tr>
</table>

### 모드 상세

| 모드 | 흐름 | 종료 조건 | 프롬프트 톤 |
|:-----|:-----|:----------|:------------|
| **1 Round** | 초안 → 검토 → 판단 → 최종 | 1회 검토 후 | 건설적 피드백 |
| **2 Round** | 초안 → 검토 → 수정 → 재검토 → 최종 | 2회 검토 후 | 건설적 피드백 |
| **Adaptive** | 2 Round 패턴 반복 | 합의 도달 또는 최대 5라운드 | 건설적 + 합의 판단 |
| **Devil's Advocate** | 주장 → 반박 → 재반박 → ... | 명시적 패배 선언 또는 최대 10라운드 (연장 가능) | 비판적 반론 |

> **Devil's Advocate** 모드는 토론자 대 토론자 구조입니다. 각 측은 상대의 논리적 약점과 근거 없는 주장을 공격합니다. 한쪽이 *"패배를 인정합니다"* 라고 명시적으로 선언할 때까지 토론이 계속됩니다. 10라운드 도달 시 사용자에게 계속 여부를 확인합니다.

---

## 사전 요구사항

| 요구사항 | 상세 |
|:---------|:-----|
| **Claude Code** | [Anthropic 공식 CLI](https://docs.anthropic.com/en/docs/claude-code) |

이것이 전부입니다. 외부 CLI도, Python도, 추가 패키지도 필요 없습니다. C2C Collab은 Claude Code의 내장 Agent tool을 사용합니다.

---

## 설치 방법

### 스킬 클론 & 설치

```
git clone https://github.com/dbaek-star/C2C.git
```

#### Windows — Git Bash (권장)

> Windows에서 Claude Code는 Git Bash를 기본 셸로 사용합니다. 이 방법을 권장합니다.

```bash
mkdir -p ~/.claude/skills/c2c-collab
cp -r C2C/SKILL.md C2C/references ~/.claude/skills/c2c-collab/
```

#### Windows — CMD

```cmd
mkdir "%USERPROFILE%\.claude\skills\c2c-collab"
xcopy /E /I /Y "C2C\SKILL.md" "%USERPROFILE%\.claude\skills\c2c-collab\"
xcopy /E /I /Y "C2C\references" "%USERPROFILE%\.claude\skills\c2c-collab\references"
```

#### Windows — PowerShell

```powershell
$dest = "$env:USERPROFILE\.claude\skills\c2c-collab"
New-Item -ItemType Directory -Force -Path $dest | Out-Null
Copy-Item -Path ".\C2C\SKILL.md" -Destination $dest
Copy-Item -Path ".\C2C\references" -Destination $dest -Recurse -Force
```

#### macOS / Linux

```bash
mkdir -p ~/.claude/skills/c2c-collab
cp -r C2C/SKILL.md C2C/references ~/.claude/skills/c2c-collab/
```

### 설치 확인

Claude Code를 열고 트리거 문구를 입력합니다:

```
> C2C 협업으로 프로젝트 계획 세워줘
```

스킬이 로드되면 설치 완료입니다!

---

## 사용 방법

### 트리거 문구

다음 문구로 스킬을 실행할 수 있습니다 (한국어 및 영어):

| 언어 | 트리거 예시 |
|:-----|:------------|
| 한국어 | `"C2C 협업"`, `"클로드 협업"`, `"서브에이전트와 협업"`, `"서브에이전트한테 물어봐"`, `"AI끼리 토론"`, `"두 AI 의견 비교"` |
| 영어 | `"C2C collab"`, `"collaborate with subagent"` |

### 인터랙티브 설정

스킬이 실행되면 두 개의 선택 프롬프트가 동시에 표시됩니다:

```
┌─ 협업 모드 ────────────────────────────────────┐
│  ● 2 Round (추천)                              │
│  ○ 1 Round                                     │
│  ○ Adaptive Round                              │
│  ○ Devil's Advocate                             │
└────────────────────────────────────────────────┘

┌─ 서브에이전트 모델 ───────────────────────────┐
│  ● Claude Sonnet (추천)                        │
│  ○ Claude Opus                                 │
│  ○ Claude Haiku                                │
└────────────────────────────────────────────────┘
```

### 세션 예시

```bash
# Claude Code에서
> 서브에이전트와 협업해서 이커머스 플랫폼의 마이크로서비스 아키텍처 설계해줘

# Claude가 다음을 수행합니다:
# 1. 모드와 모델 선택 요청
# 2. 웹 검색으로 주제 조사
# 3. 초안 작성
# 4. 서브에이전트(Claude-B) 생성하여 독립적 검토
# 5. 피드백 평가 후 최종 산출물 생성
```

---

## 모델 지원

### 사용 가능한 모델

| 모델 | Agent `model` | 적합한 용도 |
|:------|:-------------|:------------|
| **Claude Sonnet** | `"sonnet"` | 빠르고 균형잡힌 성능 — 대부분의 작업에 추천 |
| **Claude Opus** | `"opus"` | 복잡한 추론 & 깊은 분석 |
| **Claude Haiku** | `"haiku"` | 경량, 빠른 응답 |

> 서브에이전트 모델은 `CLAUDE_CODE_SUBAGENT_MODEL` 환경변수로 오버라이드할 수 있습니다.

---

## 산출물 구조

모든 산출물은 현재 작업 디렉토리 하위에 저장됩니다:

```
{CWD}/.c2c/collab/{YYYYMMDD_HHMMSS}_{주제}/
├── round1_1_claude_draft.md          # Claude-A의 초안
├── round1_2_claude-b_review.md       # Claude-B의 검토
├── round1_3_claude_decision.md       # Claude-A의 피드백 판단
├── round2_1_claude-b_review.md       # (2 Round+) Claude-B의 재검토
├── round2_2_claude_decision.md       # (2 Round+) Claude-A의 최종 판단
├── collab_final.md                   # 최종 협업 산출물
└── collab_summary.md                 # 협업 요약 & 메타데이터
```

### 요약 보고서 (`collab_summary.md`)

요약에 포함되는 항목:
- 사용된 서브에이전트 모델
- 협업 모드 & 총 라운드 수
- 총 서브에이전트 호출 횟수
- 원래 요청 요약
- 라운드별 주요 결정 사항 (수용/반박 항목과 사유)
- 최종 산출물 요약
- 산출물 파일 위치

---

## 아키텍처

```
C2C/
├── SKILL.md                     # 스킬 정의 & 오케스트레이션 규칙
└── references/
    ├── subagent-common.md       # 서브에이전트 호출 공통 규칙 & 파라미터
    └── modes.md                 # 협업 모드 상세 명세
```

### 핵심 설계 결정

| 결정 | 이유 |
|:-----|:-----|
| **외부 의존성 제로** | Claude Code 내장 Agent tool 사용 — CLI, 스크립트, 패키지 불필요 |
| **고정 프롬프트 미사용** | 매 서브에이전트 호출마다 주제, 라운드, 모드, 이전 피드백에 기반한 동적 프롬프트 생성 |
| **독립 컨텍스트** | 서브에이전트가 격리된 컨텍스트에서 실행되어 진정한 독립 평가 보장 |
| **솔직한 평가** | 강제 비판 할당량 없음 — 리뷰어가 동의할 부분은 동의, 문제점은 지적 |
| **컨텍스트 관리** | Round 3+에서는 요약 컨텍스트 + Read tool 참조로 Context Bloat 방지 |

---

## 사용 예시

### 시스템 아키텍처 설계

```
> C2C 협업으로 실시간 알림 시스템 설계해줘
> 모드: 2 Round | 모델: Claude Sonnet

결과: Claude-A 아키텍처 초안 → Claude-B 확장성 문제 지적
→ Claude-A 이벤트 기반 설계로 수정 → Claude-B 검증 → 최종 산출물
```

### 기술 문서 작성

```
> 서브에이전트와 함께 API 설계 문서 작성해줘
> 모드: Adaptive | 모델: Claude Opus

결과: 엔드포인트 설계, 에러 처리 패턴, 인증 흐름에 대해
양쪽 인스턴스가 합의할 때까지 반복 개선
```

### 기술적 의사결정 토론

```
> AI끼리 토론해봐: 우리 스타트업에 마이크로서비스 vs 모놀리스 중 뭐가 좋을까?
> 모드: Devil's Advocate | 모델: Claude Sonnet

결과: Claude-A가 모놀리스 주장 (단순성, 속도)
↔ Claude-B가 마이크로서비스 주장 (확장성, 팀 독립성)
→ 한쪽이 상대의 논거를 반박할 수 없을 때 패배 인정
```

---

## Gemini Collab에서 마이그레이션

이전에 Gemini Collab을 사용했다면, 기존 `.gemini/context.md`가 첫 실행 시 자동으로 `.c2c/context.md`로 마이그레이션됩니다. `.gemini/collab/`에 있는 이전 협업 산출물은 읽기 전용 참조로 계속 접근 가능합니다.

---

## 기여하기

기여를 환영합니다! 다음 절차를 따라주세요:

1. 이 저장소를 **Fork**합니다
2. 기능 브랜치를 **생성**합니다 (`git checkout -b feature/amazing-feature`)
3. 변경사항을 **커밋**합니다 (`git commit -m 'Add amazing feature'`)
4. 브랜치에 **Push**합니다 (`git push origin feature/amazing-feature`)
5. **Pull Request**를 생성합니다

### 기여 가능한 영역

- 새로운 협업 모드 추가
- 트리거 문구의 다국어 지원 확대
- 요약 보고서 포맷 개선
- 다른 서브에이전트 타입 연동
- 테스트 커버리지 확대

---

## 라이선스

이 프로젝트는 MIT 라이선스에 따라 배포됩니다 — 자세한 내용은 [LICENSE](LICENSE) 파일을 확인하세요.

---

<div align="center">

**멀티 에이전트 AI 협업 시대를 위해 만들어졌습니다**

<br/>

<img src="https://img.shields.io/badge/Claude--A-Orchestrator-7C3AED?style=for-the-badge" alt="Claude-A">
<img src="https://img.shields.io/badge/Claude--B-Subagent-2563EB?style=for-the-badge" alt="Claude-B">

<br/><br/>

*두 개의 머리가 하나보다 낫다 — 같은 모델이라 해도.*

</div>
