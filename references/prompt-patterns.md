# Prompt Patterns Reference — Claude

This file contains reusable structural patterns for generating high-quality Claude prompts. Referenced by the prompt-creator skill during generation. It is the Claude counterpart to `gemini-guide.md` and `gpt-guide.md`.

**Last verified: September 2026.** Model names and API behaviors below reflect Anthropic documentation as of 2026-09-12. Anthropic ships models frequently — re-verify before pinning a model name or claiming an API parameter errors.

---

## Model landscape (verified September 2026)

| Model | API ID | Positioning |
|---|---|---|
| Claude Fable 5.1 | `claude-fable-5-1` | Most capable widely released model — demanding reasoning, long-horizon agentic work |
| Claude Opus 5 | `claude-opus-5` | Flagship general-purpose tier |
| Claude Sonnet 5 | `claude-sonnet-5` | Balanced everyday tier |
| Claude Haiku 4.5 | `claude-haiku-4-5` | Fastest, cheapest — subagents, high-volume work |

Opus 4.8 / 4.7 / 4.6 and Sonnet 4.6 remain served as the previous generation. Current models carry a 1M token context (Haiku 4.5: 200K) and support up to 128K output tokens.

---

## API-level behavior that changes how you write the prompt

Claude's prompting conventions shifted with the 4.6+ generation. These are the changes that most often make an older prompt wrong rather than merely verbose:

| Area | Current behavior |
|---|---|
| **Thinking** | `thinking: {type: "adaptive"}` — the model decides when and how deeply to think. On Claude Fable 5.1 thinking is always on. The fixed `budget_tokens` concept is gone: it returns a 400 on Fable 5/5.1, Opus 5/4.8/4.7 and Sonnet 5. Haiku 4.5 still uses `budget_tokens`. |
| **Effort** | `output_config: {effort: ...}` with `low` / `medium` / `high` / `xhigh` / `max` controls reasoning depth and token spend. This is the dial to reach for instead of prose about thoroughness. |
| **Sampling** | `temperature`, `top_p`, `top_k` are removed on Fable 5/5.1, Opus 5/4.8/4.7 and Sonnet 5 — they return a 400. Do not write prompts or integration code that set them. |
| **Structured output** | `output_config: {format: {...}}`. The older `output_format` parameter is deprecated. |
| **Assistant prefill** | Removed — a trailing assistant turn returns a 400 on current models. Use structured outputs or a system-prompt format instruction instead. |

**The practical consequence for prompt writing:** depth, length and rigor are now set by *configuration*, and the prompt carries *context and intent*. A prompt that spends its words instructing Claude to be thorough, to plan, or to reason step by step is spending them on behavior the model already has — and, on current models, over-prescription measurably degrades output quality.

---

## Dated patterns — do not generate these for current Claude models

These were correct for earlier Claude generations and are now counterproductive:

| Dated pattern | What to do instead |
|---|---|
| `<thinking>` / `<answer>` scaffolds, "think step by step", `<scratchpad>` blocks | Adaptive thinking plus `effort`. On Claude Fable 5.1, instructing the model to reproduce its reasoning can trigger a refusal (reasoning extraction). |
| "Be thorough. Do not be lazy. Do not stop early." | Delete — current models are proactive by default. |
| `CRITICAL:` / `You MUST` / `NEVER EVER` stacked across a prompt | State the one or two real constraints plainly, with their reason. When everything is critical, nothing is. |
| `STEP 1: … STEP 2: …` choreography for judgment tasks | State the outcome, the constraints, and how to verify. Keep numbered steps only where order genuinely matters. |
| Numeric output ceilings ("at most 150 words", "exactly 5 bullets") | Qualitative guidance tied to audience and purpose. Hard caps starve reasoning on difficult problems. |
| Long prohibition lists | Describe success. A prohibition against a failure the model wasn't going to make can anchor it toward that failure. |
| Assistant prefill + stop sequences + "output ONLY valid JSON" + retry-on-parse | Structured outputs (`output_config.format`). |

---

## Pattern 1 — Standard task prompt (API system prompt)

```xml
<role>
You are a [specific role] with expertise in [domain]. Your purpose is to [primary function].
</role>

<context>
You are working with [audience description]. They expect [tone/style]. The output will be used for [purpose].
</context>

<instructions>
1. [First instruction — most important behavior]
2. [Second instruction]
3. [Additional instructions as needed]

When handling [specific scenario], do the following:
- [Sub-instruction]
- [Sub-instruction]
</instructions>

<constraints>
- [What to avoid, stated constructively: "Keep responses under 200 words" rather than "Don't write long responses"]
- [Behavioral boundary]
</constraints>

<examples>
<example>
<input>[Example user input]</input>
<output>[Example ideal output]</output>
</example>

<example>
<input>[Second example input]</input>
<output>[Second example output]</output>
</example>

<example>
<input>[Third example — ideally showing an edge case]</input>
<output>[Third example output]</output>
</example>
</examples>

<output_format>
[Description of the expected output structure: JSON schema, markdown template, plain text conventions, etc.]
</output_format>
```

---

## Pattern 2 — Analytical / reasoning prompt

Use this when the task involves analysis, classification, decision-making, or multi-step reasoning.

**Do not write a chain-of-thought scaffold.** On current Claude models, reasoning depth is set with adaptive thinking plus `output_config.effort`, not with prompt text. The prompt's job is to state the task, the criteria that should drive the judgment, and how to verify the result — then get out of the way.

```xml
<role>
You are a [role] specializing in [domain].
</role>

<context>
[Background information relevant to the task]
</context>

<instructions>
[The analytical task, stated directly.]

Base your judgment on these criteria:
- [Criterion 1 — what makes an answer right here]
- [Criterion 2]
- [Criterion 3]

Where the input is ambiguous or information is missing, say so explicitly rather than filling the gap with an assumption.
</instructions>

<output_format>
[Specify the structure of the answer]
</output_format>
```

Set `effort` to match the difficulty: `low` for routine classification, `high` for genuine analysis, `max` when correctness matters more than cost. Read the reasoning through the API's thinking blocks if you need to inspect it.

### Legacy variant — visible reasoning as a deliverable

Ask for reasoning *in the output* only when the trace itself is the product — an auditable rationale a human must review, a teaching artifact, or a destination that surfaces no thinking blocks. Even then, name it for what it is rather than dressing it as a thinking aid:

```xml
<instructions>
[The analytical task]

Structure your response in two parts:

**Rationale** — the criteria that drove your conclusion and how you weighed them, in a few sentences.
**Conclusion** — [the required format].
</instructions>
```

Avoid `<thinking>` and `<answer>` tag names here: on Claude Fable 5.1, instructing the model to reproduce its internal reasoning can trigger a refusal.

---

## Pattern 3 — Document processing prompt

Use this when the task involves analyzing, summarizing, or extracting from long documents.

```xml
<document>
{{DOCUMENT_CONTENT}}
</document>

<role>
You are a [role] tasked with [specific document task].
</role>

<instructions>
Using the document provided above, perform the following:

1. [Primary extraction/analysis task]
2. [Secondary task if applicable]
3. [Quality check or verification step]

Important considerations:
- Reference specific sections or quotes from the document to support your output.
- If information is missing or ambiguous in the document, note this explicitly rather than guessing.
- [Additional domain-specific instructions]
</instructions>

<output_format>
[Specify structure: summary format, extraction schema, analysis template]
</output_format>
```

---

## Pattern 4 — Conversational assistant (claude.ai)

Use this for prompts intended for direct claude.ai conversations. Lighter structure, more natural tone.

```
You are a [role] helping [audience] with [domain].

Your approach:
- [Key behavior 1 — e.g., "Start by understanding what the user needs before jumping to solutions"]
- [Key behavior 2]
- [Key behavior 3]

Style guidelines:
- [Tone instruction — e.g., "Speak like a knowledgeable colleague, not a textbook"]
- [Length instruction — e.g., "Keep answers concise unless the user asks for detail"]
- [Format instruction — e.g., "Use bullet points for lists, but write explanations in prose"]

When you're unsure about something, say so directly rather than guessing.

[Optional: 1-2 short examples of ideal exchanges if the task has a specific format]
```

---

## Pattern 5 — Classification / routing prompt

Use this when Claude needs to categorize inputs and respond differently based on category.

```xml
<role>
You are a [role] that classifies incoming [items] and responds accordingly.
</role>

<categories>
<category name="[Category A]">
  <description>[When this category applies]</description>
  <response_approach>[How to handle this category]</response_approach>
</category>

<category name="[Category B]">
  <description>[When this category applies]</description>
  <response_approach>[How to handle this category]</response_approach>
</category>

<category name="[Category C — fallback]">
  <description>[When no other category fits]</description>
  <response_approach>[How to handle edge cases]</response_approach>
</category>
</categories>

<instructions>
1. Read the input.
2. Determine which category best fits.
3. Apply the corresponding response approach.
4. If the input spans multiple categories, [specify how to handle: pick primary, address both, etc.].
</instructions>

<examples>
[3-5 examples covering different categories, including at least one edge case]
</examples>
```

---

## Few-shot example guidelines

When including examples in a prompt:

- **Minimum 3 examples** for tasks with a specific format or tone to match.
- **Include at least one edge case** (unusual input, boundary condition, ambiguous case).
- **Show the exact output format** — don't describe it, demonstrate it.
- **Keep examples representative** — they should cover the range of likely inputs, not just the easy cases.
- **Order examples** from simple to complex.
- **Vary them deliberately.** Examples are the strongest signal in a prompt — Claude matches their length, tone and structure. A single gold output freezes that exact shape into every response.
- **Skip examples for judgment the model already owns.** Keep them where they pin a genuinely format-sensitive output shape; drop them where they only demonstrate competence.

---

## XML tag conventions

| Tag | Purpose |
|---|---|
| `<role>` | Who Claude is in this context |
| `<context>` | Background information, audience, purpose |
| `<instructions>` | What Claude should do, step by step |
| `<constraints>` | Boundaries, limitations, things to avoid |
| `<examples>` | Input/output demonstrations |
| `<example>` | Single example wrapper |
| `<input>` | Example input within an example |
| `<output>` | Example ideal output within an example |
| ~~`<thinking>`~~ | **Retired** — reasoning depth is set by adaptive thinking + `effort`, not by tags. Can trigger a refusal on Claude Fable 5.1 |
| ~~`<answer>`~~ | **Retired** — no longer needed once no thinking block precedes it |
| `<output_format>` | Description of expected output structure |
| `<document>` | Long-form content to process |
| `<categories>` | Classification categories |
| `<category>` | Single category definition |

---

## Tone calibration reference

| Tone keyword | What it means in practice |
|---|---|
| Formal | Complete sentences, no contractions, precise vocabulary, structured output |
| Professional | Clear and polished, contractions acceptable, focused on efficiency |
| Conversational | Natural speech patterns, casual but competent, uses "you" and "I" freely |
| Pedagogical | Explains reasoning, builds understanding progressively, uses analogies |
| Technical | Domain-specific vocabulary, assumes reader expertise, dense information |
| Friendly | Warm, encouraging, uses light humor if appropriate, reassuring |
| Neutral | No personality, factual, minimal style — lets the content speak |
