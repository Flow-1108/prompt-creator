# Prompt Creator — Claude Skill (v4)

A Claude Code skill that walks you through creating optimized prompts for **Claude**, **Gemini**, or **GPT**, whatever the task.

*Documentation en français : [README.fr.md](./README.fr.md)*

> **Content verified September 2026.** Model names and API parameters move fast across all three providers. Each reference file carries its own verification date — if it is more than a couple of months old, re-check before relying on a specific model name.

---

## What this skill does

When triggered, the skill **first asks which AI the prompt is for** (Claude, Gemini, or GPT), then runs a natural conversation to gather context, and produces **a single final prompt** — no explanation, no commentary, ready to copy and paste.

The generated prompt adapts to:
- **The target AI**: Claude (Anthropic best practices), Gemini (Google's PTCF framework), or GPT (CTCO framework)
- **The destination**: a conversation UI (claude.ai / AI Studio / ChatGPT) or an API system prompt

---

## The principle behind v4

All three providers have converged on the same architecture: **reasoning depth is now a parameter, not prompt text.**

| Provider | Depth control |
|---|---|
| Anthropic | `thinking: {type: "adaptive"}` + `output_config.effort` (`low` → `max`) |
| OpenAI | `reasoning_effort` (`none` → `max`) |
| Google | `thinking_level` (`low` / `medium` / `high`) |

What follows from that:

- **No chain-of-thought scaffolding.** "Think step by step", `<thinking>` tags, and planning choreography duplicate reasoning the model already does internally, and can make output **worse**.
- **No sampling parameters.** Claude's frontier models reject `temperature`/`top_p`/`top_k`; Gemini 3+ instructs developers to remove them from generation configs.
- **Structured output is a first-class API feature** on all three. Prompt-level JSON scaffolding (prefill, stop sequences, "output ONLY valid JSON", retry-on-parse) is obsolete.
- **Instruction-following became literal.** Inflated emphasis over-triggers, and leftover hedges ("try to", "if possible") read as permission to under-deliver.

The prompt's job is now to carry **context and intent** — audience, product, quality bar, constraints, and the reasons behind them. That is what only the author knows.

---

## Current models (verified September 2026)

| Provider | Models |
|---|---|
| **Anthropic** | Claude Fable 5.1 (`claude-fable-5-1`), Opus 5 (`claude-opus-5`), Sonnet 5 (`claude-sonnet-5`), Haiku 4.5 (`claude-haiku-4-5`) |
| **OpenAI** | GPT-6 Astra (`gpt-6-astra`), GPT-6 Sol (`gpt-6-sol`), GPT-6 Luna (`gpt-6-luna`), GPT-5.6 Terra (`gpt-5.6-terra` — no GPT-6 Terra exists) |
| **Google** | Gemini 3.8 / 3.7 / 3.6 / 3.5 Flash, 3.5 Flash-Lite, 3.1 Pro (preview) |

`o3` is no longer in OpenAI's catalog. The meaningful distinction is no longer *which model reasons* but *at what effort level*.

---

## Phrases that trigger the skill

- "create a prompt for me"
- "write a system prompt"
- "improve my prompt"
- "generate instructions for Claude / Gemini / GPT"
- "help me write a prompt for…"
- "I need a good prompt for…"
- "prompt for my app"
- "system prompt for Gemini"
- "prompt for ChatGPT"
- *(and French equivalents)*

---

## Folder structure

```
prompt-creator/
├── SKILL.md                        # Main skill file (v4)
├── README.md                       # This documentation (English)
├── README.fr.md                    # French documentation
└── references/
    ├── prompt-patterns.md          # Claude guide: models, patterns, dated patterns
    ├── gemini-guide.md             # Gemini guide: PTCF, thinking_level, patterns
    └── gpt-guide.md                # GPT guide: CTCO, reasoning_effort, agents
```

---

## How it works

### 0. Target AI selection
Always the very first question: **"Is this prompt for Claude, Gemini, or GPT?"** That choice governs the structure, patterns, and conventions used for generation.

### 0.5. Skill recommendation *(Claude + Claude Code only)*
After learning the objective, the skill checks whether an available Claude Code skill covers the task. **It reads the skill list from the current session** rather than a hardcoded catalog — the installed set differs per machine, per marketplace, and per user, and grows with every release. If it finds a match, it **mentions it** and continues building the prompt, **optimized for that skill** — the two are complementary, not alternatives. Otherwise this phase stays silent.

> Example: the user wants to process PDFs → the skill mentions the `pdf` skill if available, and generates a prompt calibrated to use it effectively.

### 1. Context gathering
The skill asks questions **one at a time**, in a logical order, without technical jargon. It covers at minimum:

| Question | Why |
|----------|-----|
| Target AI | Pick the right best practices (Claude / Gemini / GPT) |
| Objective | Know what the prompt must accomplish |
| Role / persona | Calibrate tone and expertise |
| Audience | Adapt level and register |
| Tone | Formal, pedagogical, conversational… |
| Destination | Conversation UI or API system prompt |
| Constraints | What must be avoided |
| Input/output examples | For tasks with a precise format |
| Output format | List, JSON, free text, table… |
| Multimodal inputs *(Gemini/GPT)* | Images, video, audio to process? |
| Long context *(Gemini/GPT)* | Large documents in context? |
| Agent behavior | Autonomous tool-using agent, or standard assistant? |
| Reasoning depth | Complex analysis or fast execution? Maps to a parameter on all three providers |
| Specific model *(API, only if it changes something)* | A cheaper tier may need more explicit scaffolding than a flagship |

### 2. Clarification
The skill keeps asking while it detects **ambiguity or missing information**. It decides for itself when it has enough context.

### 3. Prompt generation
Once context is sufficient, the skill produces the final prompt applying the **target AI's best practices**:

**For Claude (Anthropic):**
- Semantic XML tags: `<role>`, `<context>`, `<instructions>`, `<examples>`, `<output_format>`
- Few-shot examples (3-5), deliberately varied — Claude matches their length, tone, and structure
- **No reasoning scaffold**: depth comes from `effort`, not `<thinking>` tags. On Claude Fable 5.1, instructing the model to reproduce its reasoning can trigger a refusal
- Long documents at the top of the prompt
- Calm, direct tone — no `SHOUTING IN CAPS`
- No prefill or prompt-level JSON scaffolding: `output_config.format` handles it

**For Gemini (Google):**
- PTCF framework: Persona · Task · Context · Format
- Few-shot examples as a first-line quality lever (strong Google recommendation)
- **`thinking_level`** (`low` / `medium` / `high`) instead of planning instructions. `minimal` returns an error
- **No sampling parameters**: `temperature`, `top_p`, `top_k`, and `candidate_count` must be removed on Gemini 3+
- Positive constraints (broad negatives disrupt Gemini)
- Consistent delimiters: XML *or* Markdown, never mixed
- Native multimodal support, context up to 1M tokens

**For GPT (OpenAI):**
- CTCO framework: Context → Task → Constraints → Output
- Constraints separated from the task (reduces instruction drift)
- 3 agent instructions: Persistence + Scope/completion + Tool usage
- Negative instructions always paired with a positive alternative
- `reasoning_effort`: `none`/`low` for extraction and formatting, `medium` as default, `high`/`xhigh`/`max` for hard problems
- Structured Outputs: token-level JSON constraint via `response_format`
- Prompt caching: static content at the top, dynamic at the bottom
- GPT-6 specifics (Astra, Sol, Luna): bias it toward action when intent is clear, ask explicitly for prose (it defaults to lists), scope test thoroughness on code tasks

---

## Prompt engineering principles applied

### Common to all three
- **Component structure** — clear sections with XML tags
- **Few-shot examples** — 3-5 for tasks with an expected format
- **Calibrated tone** — direct language, positive constraints, emphasis earned rather than default
- **Depth in configuration** — never "think step by step" in the prompt text
- **Say it once** — leaner prompts measurably outperform padded ones

### Claude-specific
- **Adaptive thinking + `effort`** instead of reasoning tags
- Rich context and explicit behaviors; no step-by-step choreography for judgment tasks

### Gemini-specific
- **PTCF framework** (Google's recommended structure)
- **`thinking_level`** instead of explicit planning
- **Concision** — Gemini 3 follows instructions well without over-specification
- **No sampling parameters** — `temperature` should be absent, not tuned
- **Native multimodal** — specific questions about media, not "analyze this"

### GPT-specific
- **CTCO framework** — a reliable convention aligned with OpenAI's docs (see caveat below)
- **Isolated constraints** — dedicated section, separate from the task
- **3 agent instructions** — persistence, scope/completion, tool usage
- **No explicit CoT** — every current flagship reasons internally
- **Caching** — static instructions at the top, variable content at the bottom
- **Double placement** — instructions before AND after long documents

---

## Unverified claims

Two statements are explicitly flagged as unconfirmed in the reference files:

- **CTCO** is a widely used community convention consistent with OpenAI's documented advice, but it was not found as a named framework in OpenAI's own documentation (checked 2026-09-12). Earlier versions of this skill presented it as "OpenAI's Official Structure".
- **GPT-6 Astra's `reasoning_effort` range** differs between two official OpenAI pages (`low`/`medium`/`high` on one, up to `max` on the other). Both agree `none` is unsupported.

---

## Installation

Place the skill folder in your Claude Code plugins directory:

```
~/.claude/plugins/<your-plugin>/skills/prompt-creator/
```

Or directly in:

```
~/.claude/skills/prompt-creator/
```

---

## Compatibility

| Environment | Supported |
|-------------|-----------|
| claude.ai | Yes |
| Claude Code | Yes |
| API (system prompt) | Yes — the skill can generate prompts for this target |

---

## Reference files

**[`references/prompt-patterns.md`](./references/prompt-patterns.md)** — Claude guide:
1. Model landscape and the API behaviors that change how you write prompts
2. Dated patterns to stop generating
3. 5 prompt structures (standard task, analytical, document processing, conversational, classification)
4. XML tag conventions

**[`references/gemini-guide.md`](./references/gemini-guide.md)** — Gemini guide:
1. Model landscape and Gemini 3+ generation config
2. PTCF framework and template
3. 6 prompt patterns (standard, analytical, documents, conversational, multimodal, JSON)
4. Best practices, pitfalls, XML tag conventions

**[`references/gpt-guide.md`](./references/gpt-guide.md)** — GPT/OpenAI guide:
1. Model landscape and `reasoning_effort` levels
2. GPT-6 family behavioral notes
3. CTCO framework and template
4. 6 prompt patterns (standard, reasoning, agent, long documents, conversational, JSON)
5. Best practices, pitfalls, XML conventions

---

## Author

Built with [Claude Code](https://claude.ai/code) · [Flow-1108](https://github.com/Flow-1108)
