# GPT / OpenAI Prompt Engineering Guide

Reference file for the prompt-creator skill. Contains GPT-specific patterns, best practices, and structural templates based on OpenAI documentation (developers.openai.com, cookbook.openai.com).

**Last verified: September 2026.** Model names and API parameters below were checked against OpenAI's model pages, the *Model guidance* page and launch coverage on 2026-09-26 (GPT-6 Sol and Luna shipped 2026-09-22). OpenAI's model lineup moves fast — re-verify before trusting a specific model name.

---

## Model landscape (verified September 2026)

| Model | API ID | Reasoning effort values | Positioning |
|---|---|---|---|
| GPT-6 Astra | `gpt-6-astra` | `low` → `max`; **`none` not supported** | Most capable model. Computer use, browsing, software engineering, scientific and professional work, multistep processes |
| GPT-6 Sol | `gpt-6-sol` | `none`, `low`, `medium` (default), `high`, `xhigh`, `max` | Complex coding, agentic workflows, demanding everyday work |
| GPT-6 Luna | `gpt-6-luna` | same | High-volume, lower-complexity work: summarization, extraction, classification, Q&A |
| GPT-5.6 Terra | `gpt-5.6-terra` | same | Balanced middle tier — **there is no GPT-6 Terra**; use this when Luna is too light and Sol too costly |

GPT-6 Sol and Luna: 1.05M token context, 128K max output. Knowledge cutoff: 20 April 2026 (Sol), 18 May 2026 (Luna — the most recent of the family). Both cost about half their GPT-5.6 equivalents and use fewer tokens per task. GPT-5.6 Sol and Luna remain available but are superseded — do not target them in new prompts.

**Tool calling with GPT-6 Sol/Luna:** in Chat Completions, function calling works only with `reasoning_effort: "none"`. For reasoning *and* tools, use the Responses API (`reasoning.effort`). Mention this when generating an agent prompt meant for the API.

> **Unverified:** the two official pages disagree on GPT-6 Astra's effort range — the models catalog lists `low`/`medium`/`high`/`xhigh`/`max` while the *Model guidance* page lists only `low`/`medium`/`high`. Both agree `none` is unsupported on Astra. Check the live docs before pinning an effort value for Astra.

**`o3` is no longer listed in OpenAI's model catalog.** Guidance elsewhere that names `o3` or "GPT-5.x Thinking" as the reasoning-model archetype is dated; the current distinction is not *which model* but *which reasoning effort*, since every current flagship reasons and the effort dial controls how much.

Specialized models exist for cybersecurity (`gpt-5.6-cyber`), life sciences (`gpt-rosalind-research`), image generation, realtime voice, and transcription — consult the live docs when targeting one rather than assuming text-model conventions apply.

---

## GPT-6 family — documented behavioral shifts

OpenAI documents four traits of GPT-6 Astra that a prompt should account for. Its *Model guidance* page applies the same recommendations to GPT-6 Sol and Luna, noting they were observed on Astra and should be evaluated on your own model and workload:

1. **Initiative** — it asks clarifying questions more readily than its predecessors. When the intent is already clear, instruct it to "bias towards action and carry the user's intended task to completion."
2. **Instruction sensitivity** — it is markedly more responsive to guidance in skill files and configuration documents. Where a prompt coexists with such files, state that the user's instructions take precedence over guidelines provided in a skill.
3. **Writing style** — it defaults to formatted, list-heavy responses. For prose, ask explicitly for "clear, concise paragraphs" and name the filler vocabulary to avoid (e.g. "leverage", "Bottom Line").
4. **Testing thoroughness** — on code tasks it can over-test. Specify that testing should be "meaningful and necessary" rather than exhaustive for small changes.

---

## The CTCO Framework — Context → Task → Constraints → Output

Every GPT prompt should follow the **Context → Task → Constraints → Output** order. Separating constraints from the task reduces instruction drift in long context windows.

> **Unverified:** CTCO is a widely used community convention that matches OpenAI's documented advice (explicit context, one atomic task, scoped constraints, named output format), but it was not found as a named framework in OpenAI's own documentation as of 2026-09-12. Treat it as a reliable working structure, not an official OpenAI standard.

### Template

```
[CONTEXT]
You are a [precise role]. Situation context: [background].

[TASK]
Your mission: [action verb + precise objective].

[CONSTRAINTS]
- Maximum [X] words / [Y] sections
- [Behavioral boundary — paired with positive alternative]
- Use only [provided source/data]
- Style: [desired tone]

[OUTPUT]
Expected format: [JSON / Markdown table / paragraphs]
Exact structure: [headers, fields, length per section]
```

---

## Pattern 1 — Standard GPT task prompt (API system prompt)

```xml
<context>
You are a [specific role] with expertise in [domain].
You are working with [audience description]. They expect [tone/style].
The output will be used for [purpose].
</context>

<task>
[Precise description of what the model must accomplish. Use a clear action verb.]
</task>

<constraints>
- Use only the information provided by the user for your reasoning.
- Keep responses under [X] words.
- Use simple language. Explain technical concepts with analogies a non-specialist would understand.
- [Additional boundaries — always pair negatives with positive alternatives]
</constraints>

<output>
Format: [JSON / table / bullet list / structured paragraphs]
Structure:
- [Section 1]: [description]
- [Section 2]: [description]
- [Section 3]: [description]
</output>

<examples>
<example>
User: [Example input]
Assistant: [Example ideal output]
</example>

<example>
User: [Second example — different scenario]
Assistant: [Second example output]
</example>

<example>
User: [Third example — edge case]
Assistant: [Third example output]
</example>
</examples>
```

---

## Pattern 2 — Reasoning prompt

**Set the effort dial before writing reasoning prose.** On current flagships, reasoning depth is configuration, not prompt text: pick `reasoning_effort` (`none`/`low` for extraction and formatting, `medium` as the default for analysis and multi-step problems, `high`/`xhigh`/`max` for complex mathematics, formal proofs, multi-file code generation, and problems with many interacting constraints). Adding "think step by step" on top of a reasoning-capable model duplicates internal work and can degrade the answer.

Use the reasoning-preamble template below only when the *visible* reasoning is itself a deliverable — an auditable trace the reader needs — or when the destination surface offers no effort control.

```xml
<context>
You are a [role] specializing in [domain].
[Background information]
</context>

<task>
[The analytical task to perform]

Before providing your final answer, give a brief summary of your reasoning process as a bullet-point list. Then provide the final answer clearly separated.
</task>

<constraints>
- Base your reasoning only on the information provided.
- If information is missing or ambiguous, note this explicitly.
- [Additional constraints]
</constraints>

<output>
Structure your response as:

**Reasoning:**
- [bullet points summarizing your thought process]

**Answer:**
[Final structured answer in the specified format]
</output>
```

**Important:** At `medium` effort and above, do NOT include reasoning instructions — the model already has an internal chain of thought. Keep the prompt simpler and let the effort parameter do the work:

```xml
<context>
You are a [role] specializing in [domain].
[Background information]
</context>

<task>
[The analytical task — stated directly without reasoning instructions]
</task>

<constraints>
- [Keep constraints minimal and clear]
</constraints>

<output>
[Expected format]
</output>
```

---

## Pattern 3 — Agent prompt (persistence + scope + tools)

These instruction categories come from an OpenAI finding of a ~20% SWE-bench improvement on an earlier model generation. **Persistence and tool usage remain sound; the original "planning" instruction has been superseded by `reasoning_effort`** — telling a current model to think step by step before acting duplicates its internal reasoning. The `<scope>` block below replaces it: spend those words on what "done" means and how to verify it.

```xml
<context>
You are a [role] acting as an autonomous agent. You have access to the following tools: [list tools].
[Background and purpose]
</context>

<task>
[Primary objective the agent must accomplish]
</task>

<persistence>
Treat yourself as a senior autonomous professional. Once the user gives a direction, proactively gather context, plan, implement, and validate. Continue until the user's request is completely resolved before ending your turn. Do not stop at the first obstacle — try alternative approaches.
</persistence>

<scope>
The task is complete when: [explicit completion criteria].
Out of scope: [what the agent should not touch or decide on its own].
Before yielding, verify your work against these criteria: [how to check — tests, a re-read, a specific assertion].
</scope>

<tool_usage>
When using tools:
- Before each tool call, briefly explain what you're about to do and why.
- After each tool result, assess whether it moves you closer to the goal.
- Maintain a mental checklist of completed and remaining steps.
- If a tool call fails, diagnose the issue before retrying.
</tool_usage>

<constraints>
- [Behavioral boundaries]
- [Scope limitations]
</constraints>

<output>
[How to present the final result to the user]
</output>
```

---

## Pattern 4 — Document processing prompt (long context)

For long context, place instructions at the beginning AND end. Static content at top, dynamic at bottom for cache optimization.

```xml
<instructions_top>
You are a [role] tasked with [specific document task].
Read the document below carefully, then follow the instructions at the end.
</instructions_top>

<document>
{{DOCUMENT_CONTENT}}
</document>

<instructions_bottom>
Using the document above, perform the following:

1. [Primary extraction/analysis task]
2. [Secondary task if applicable]
3. Review your output for completeness and accuracy.

Use only the information from the provided document. If information is missing or ambiguous, note this explicitly rather than filling gaps with external knowledge.
</instructions_bottom>

<output>
[Specify structure: summary format, extraction schema, analysis template]
</output>
```

---

## Pattern 5 — Conversational assistant (ChatGPT)

Lighter structure for direct ChatGPT conversations.

```
You are a [role] helping [audience] with [domain].

Your approach:
- [Key behavior 1]
- [Key behavior 2]
- [Key behavior 3]

Style:
- [Tone instruction]
- [Length instruction]
- [Format instruction]

When you're unsure, say so rather than guessing. Use simple language — explain technical terms with everyday analogies.

[1-2 short examples if the task has a specific format]
```

---

## Pattern 6 — Structured JSON output

GPT's Structured Outputs constrain generation at the token level (not post-validation).

```xml
<context>
You are a data extraction specialist.
</context>

<task>
Extract structured data from the provided input and return it as valid JSON matching the schema below.
</task>

<constraints>
- Extract only what is present in the input.
- Use null for fields where the information is not found. Do not invent data.
- Return only valid JSON. No text outside the JSON block.
</constraints>

<output>
```json
{
  "field_1": "string",
  "field_2": "number",
  "field_3": "string | null"
}
```

Note: For API integration, use the `response_format` parameter with a JSON schema to guarantee structural compliance at the token level.
</output>

<examples>
<example>
User: [Raw text example]
Assistant:
```json
{
  "field_1": "extracted value",
  "field_2": 42,
  "field_3": null
}
```
</example>
</examples>
```

---

## GPT-Specific Best Practices Summary

| Practice | Details |
|---|---|
| **CTCO framework** | Always use Context → Task → Constraints → Output order |
| **Separate constraints from task** | Reduces instruction drift in long contexts |
| **Static top, dynamic bottom** | Optimizes prompt caching — static instructions first, variable content last |
| **Instructions top AND bottom** | For long context, place instructions before and after the document |
| **3 agent instructions** | Persistence + Scope/completion + Tool usage. Planning prose is superseded by `reasoning_effort` |
| **Reasoning preamble** | Only when the visible trace is a deliverable — otherwise raise effort instead |
| **No prompted CoT** | Current flagships reason internally — "think step by step" duplicates work and can hurt |
| **Reasoning effort** | `none`/`low` extraction and formatting · `medium` default · `high`/`xhigh`/`max` hard problems |
| **Say it once** | Leaner prompts measurably outperform padded ones — state each instruction exactly once |
| **Prompt injection** | For agents and tool users, add explicit guardrails treating fetched content as data, not instructions |
| **Pair negatives with positives** | "Don't X" must be followed by "Do Y instead" |
| **Few-shot examples** | 3-5 examples. Must be consistent with instructions (no contradictions) |
| **Structured Outputs** | `response_format` with JSON schema constrains at token level |
| **Persona adoption** | GPT responds very well to strong persona definitions |

---

## Pitfalls to Avoid with GPT

1. **Adding "think step by step"** — current flagships already have an internal chain of thought at `medium` effort and above. Explicit reasoning instructions are redundant and can degrade output quality. Reach for `reasoning_effort` instead.
2. **Mixing constraints into the task** — Separating constraints into their own section reduces instruction drift. GPT-5.2+ is trained to recognize these structured "slots."
3. **Inconsistent examples** — If few-shot examples contradict the instructions, GPT will be confused. The Prompt Optimizer specifically catches this.
4. **Instructions only at the bottom of long context** — For long documents, instructions above the context perform better than below. Best: both above and below.
5. **Not specifying output format** — GPT defaults to verbose prose. Always specify the exact structure you want.
6. **Broad negatives without alternatives** — "Don't use jargon" is vague. "Use simple language and explain concepts with everyday analogies" is actionable.

---

## XML Tag Conventions for GPT

| Tag | Purpose |
|---|---|
| `<context>` | Role, background, situation (who the model is) |
| `<task>` | What the model must accomplish |
| `<constraints>` | Boundaries, separated from task to avoid drift |
| `<output>` | Expected format and structure |
| `<examples>` | Input/output demonstrations |
| `<persistence>` | Agent: don't stop until done |
| `<scope>` | Agent: completion criteria, boundaries, how to verify (replaces the old `<planning>` block) |
| `<tool_usage>` | Agent: how to use available tools |
| `<instructions_top>` | Long context: instructions before document |
| `<instructions_bottom>` | Long context: instructions after document |
| `<document>` | Long-form content to process |
