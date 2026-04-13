<div align="center">

# C2C Collab

### Claude-A × Claude-B — AI-to-AI Collaborative Intelligence

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-7C3AED?style=for-the-badge&logo=anthropic&logoColor=white)](https://docs.anthropic.com/en/docs/claude-code)
[![License: MIT](https://img.shields.io/badge/License-MIT-F59E0B?style=for-the-badge)](LICENSE)

<br/>

**Two Claude instances. One unified output.**
C2C Collab orchestrates structured, round-based collaboration between Claude-A (orchestrator) and Claude-B (subagent) — producing higher-quality results through independent evaluation, cross-verification, and iterative refinement.

<br/>

[한국어 README](README.ko.md)

---

<img src="https://img.shields.io/badge/Draft-Claude--A-7C3AED?style=flat-square" alt="Claude-A"> →
<img src="https://img.shields.io/badge/Review-Claude--B-2563EB?style=flat-square" alt="Claude-B"> →
<img src="https://img.shields.io/badge/Decision-Claude--A-7C3AED?style=flat-square" alt="Claude-A"> →
<img src="https://img.shields.io/badge/Final-Consensus-10B981?style=flat-square" alt="Final">

</div>

<br/>

## Table of Contents

- [Why C2C Collab?](#-why-c2c-collab)
- [How It Works](#-how-it-works)
- [Collaboration Modes](#-collaboration-modes)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Usage](#-usage)
- [Model Support](#-model-support)
- [Output Structure](#-output-structure)
- [Architecture](#-architecture)
- [Examples](#-examples)
- [Contributing](#-contributing)

---

## Why C2C Collab?

> A single AI can draft. Two AI instances can **verify, challenge, and refine**.

| Problem | Solution |
|:--------|:---------|
| Single-context blind spots | Subagent evaluates in a completely independent context |
| Unchallenged assumptions | Structured review rounds force honest evaluation |
| Hallucination risk | Web search integration + dual verification |
| Author bias | Role separation (writer vs. reviewer) provides natural diversity |

**C2C Collab** leverages Claude Code's built-in Agent tool to spawn an independent subagent — no external CLI, no scripts, zero dependencies beyond Claude Code itself.

---

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                      C2C COLLAB                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│   │ CLAUDE-A  │───▶│ CLAUDE-B │───▶│ CLAUDE-A  │        │
│   │  Draft    │    │  Review  │    │ Decision  │        │
│   │ +WebSearch│    │(subagent)│    │ Accept/   │        │
│   └──────────┘    └──────────┘    │ Reject    │        │
│                                    └─────┬────┘         │
│                                          │              │
│                              ┌───────────▼───────────┐  │
│                              │   FINAL OUTPUT        │  │
│                              │  collab_final.md      │  │
│                              │  collab_summary.md    │  │
│                              └───────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

1. **Claude-A researches** the topic via web search, then creates an initial draft
2. **Claude-B reviews** the draft independently in an isolated context (with its own web search)
3. **Claude-A evaluates** Claude-B's feedback — accepting, rejecting, or partially adopting suggestions
4. **(Optional)** Additional rounds continue until consensus or termination
5. **Final output** is generated along with a comprehensive collaboration summary

---

## Collaboration Modes

<table>
<tr>
<td width="25%" align="center">

### 1 Round
**Fast & Light**

</td>
<td width="25%" align="center">

### 2 Round
**Recommended**

</td>
<td width="25%" align="center">

### Adaptive
**Until Consensus**

</td>
<td width="25%" align="center">

### Devil's Advocate
**Debate Mode**

</td>
</tr>
<tr>
<td>

```
Claude-A Draft
    ↓
Claude-B Review
    ↓
Claude-A Decision
    ↓
  Final
```

</td>
<td>

```
Claude-A Draft
    ↓
Claude-B Review
    ↓
Claude-A Revision
    ↓
Claude-B Re-review
    ↓
Claude-A Final Call
    ↓
  Final
```

</td>
<td>

```
Claude-A Draft
    ↓
┌─── Loop ───┐
│ Claude-B Rev│
│     ↓       │
│ Claude-A Dec│
└─── × N ────┘
    ↓
  Final
```

</td>
<td>

```
Claude-A Argument
    ↓
Claude-B Counter
    ↓
Claude-A Rebuttal
    ↓
    ...
    ↓
 Surrender!
```

</td>
</tr>
<tr>
<td>

Subagent calls: **1**
Best for: Quick reviews

</td>
<td>

Subagent calls: **2**
Best for: Most tasks

</td>
<td>

Subagent calls: **1–5**
Best for: Complex topics

</td>
<td>

Subagent calls: **up to 10**
Best for: Controversial topics

</td>
</tr>
</table>

### Mode Details

| Mode | Flow | Termination | Prompt Tone |
|:-----|:-----|:------------|:------------|
| **1 Round** | Draft → Review → Decision → Final | After 1 review cycle | Constructive feedback |
| **2 Round** | Draft → Review → Revision → Re-review → Final | After 2 review cycles | Constructive feedback |
| **Adaptive** | Repeats 2-Round pattern | Consensus or max 5 rounds | Constructive + consensus judgment |
| **Devil's Advocate** | Argument → Counter → Rebuttal → ... | Explicit surrender or max 10 rounds (extendable) | Critical argumentation |

> **Devil's Advocate** mode uses a debater-vs-debater structure. Each side attacks logical weaknesses and unsupported claims. The debate continues until one side explicitly declares: *"I concede defeat."* At 10 rounds, the user is prompted to continue or stop.

---

## Prerequisites

| Requirement | Details |
|:------------|:--------|
| **Claude Code** | [Anthropic's official CLI](https://docs.anthropic.com/en/docs/claude-code) |

That's it. No external CLI, no Python, no additional packages. C2C Collab uses Claude Code's built-in Agent tool.

---

## Installation

### Clone & Install the Skill

```
git clone https://github.com/dbaek-star/C2C.git
```

#### Windows — Git Bash (Recommended)

> Claude Code on Windows uses Git Bash as the default shell. This is the recommended method.

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

### Verify

Open Claude Code and type any trigger phrase:

```
> C2C 협업으로 프로젝트 계획 세워줘
```

If the skill loads, you're all set!

---

## Usage

### Trigger Phrases

You can invoke the skill using any of these phrases (English or Korean):

| Language | Trigger Examples |
|:---------|:-----------------|
| English | `"C2C collab"`, `"collaborate with subagent"` |
| Korean | `"C2C 협업"`, `"클로드 협업"`, `"서브에이전트와 협업"`, `"서브에이전트한테 물어봐"`, `"AI끼리 토론"`, `"두 AI 의견 비교"` |

### Interactive Setup

When triggered, the skill presents two simultaneous selection prompts:

```
┌─ Collaboration Mode ───────────────────────────┐
│  ● 2 Round (Recommended)                       │
│  ○ 1 Round                                     │
│  ○ Adaptive Round                              │
│  ○ Devil's Advocate                             │
└────────────────────────────────────────────────┘

┌─ Subagent Model ──────────────────────────────┐
│  ● Claude Sonnet (Recommended)                 │
│  ○ Claude Opus                                 │
│  ○ Claude Haiku                                │
└────────────────────────────────────────────────┘
```

### Example Session

```bash
# In Claude Code
> C2C collab to design a microservices architecture for an e-commerce platform

# Claude will:
# 1. Ask you to select mode and model
# 2. Research the topic via web search
# 3. Create an initial draft
# 4. Spawn a subagent (Claude-B) for independent review
# 5. Evaluate feedback and produce final output
```

---

## Model Support

### Available Models

| Model | Agent `model` | Best For |
|:------|:-------------|:---------|
| **Claude Sonnet** | `"sonnet"` | Fast, balanced — recommended for most tasks |
| **Claude Opus** | `"opus"` | Complex reasoning & deep analysis |
| **Claude Haiku** | `"haiku"` | Lightweight, quick responses |

> The subagent model can be overridden via the `CLAUDE_CODE_SUBAGENT_MODEL` environment variable.

---

## Output Structure

All outputs are saved under your current working directory:

```
{CWD}/.c2c/collab/{YYYYMMDD_HHMMSS}_{topic}/
├── round1_1_claude_draft.md          # Claude-A's initial draft
├── round1_2_claude-b_review.md       # Claude-B's review
├── round1_3_claude_decision.md       # Claude-A's decision on feedback
├── round2_1_claude-b_review.md       # (2-Round+) Claude-B's re-review
├── round2_2_claude_decision.md       # (2-Round+) Claude-A's final decision
├── collab_final.md                   # Final collaborative output
└── collab_summary.md                 # Collaboration summary & metadata
```

### Summary Report (`collab_summary.md`)

The summary includes:
- Subagent model used
- Collaboration mode & total rounds
- Total subagent calls
- Original request summary
- Per-round key decisions (accepted/rejected items with rationale)
- Final output summary
- Output file locations

---

## Architecture

```
C2C/
├── SKILL.md                     # Skill definition & orchestration rules
└── references/
    ├── subagent-common.md       # Common subagent calling rules & parameters
    └── modes.md                 # Detailed mode specifications
```

### Key Design Decisions

| Decision | Rationale |
|:---------|:----------|
| **No external dependencies** | Uses Claude Code's built-in Agent tool — no CLI, scripts, or packages |
| **No fixed prompts** | Each subagent call gets a dynamically generated prompt based on topic, round, mode, and prior feedback |
| **Independent context** | Subagent runs in isolated context, ensuring genuine independent evaluation |
| **Honest evaluation** | No forced criticism quotas — reviewer agrees where warranted, disagrees where warranted |
| **Context management** | Round 3+ uses summarized context + Read tool references to prevent context bloat |

---

## Examples

### Planning a System Architecture

```
> C2C collab to plan a real-time notification system
> Mode: 2 Round | Model: Claude Sonnet

Result: Claude-A drafts architecture → Claude-B identifies scaling concerns
→ Claude-A revises with event-driven approach → Claude-B validates → Final output
```

### Writing a Technical Document

```
> 서브에이전트와 협업해서 API 설계 문서 작성해줘
> Mode: Adaptive | Model: Claude Opus

Result: Iterates until both instances agree on endpoint design,
error handling patterns, and authentication flow
```

### Debating a Technical Decision

```
> AI끼리 토론해봐: 우리 스타트업에 마이크로서비스 vs 모놀리스 중 뭐가 좋을까?
> Mode: Devil's Advocate | Model: Claude Sonnet

Result: Claude-A argues for monolith (simplicity, speed)
↔ Claude-B argues for microservices (scalability, team independence)
→ One side concedes when unable to counter the other's argument
```

---

## Migrating from Gemini Collab

If you previously used Gemini Collab, your existing `.gemini/context.md` will be automatically migrated to `.c2c/context.md` on first run. Previous collaboration outputs in `.gemini/collab/` remain accessible as read-only references.

---

## Contributing

Contributions are welcome! Here's how you can help:

1. **Fork** this repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

### Areas for Contribution

- New collaboration modes
- Additional language support for trigger phrases
- Enhanced summary report formatting
- Integration with other subagent types
- Test coverage

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built for the era of multi-agent AI collaboration**

<br/>

<img src="https://img.shields.io/badge/Claude--A-Orchestrator-7C3AED?style=for-the-badge" alt="Claude-A">
<img src="https://img.shields.io/badge/Claude--B-Subagent-2563EB?style=for-the-badge" alt="Claude-B">

<br/><br/>

*Two minds are better than one — even when they share the same model.*

</div>
