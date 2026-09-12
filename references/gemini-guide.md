# Gemini Prompt Engineering Guide

Reference file for the prompt-creator skill. Contains Gemini-specific patterns, best practices, and structural templates based on official Google documentation (ai.google.dev, Vertex AI).

**Last verified: September 2026.** Model names and API parameters below were checked against `ai.google.dev/gemini-api/docs/models` and `ai.google.dev/gemini-api/docs/latest-model` on 2026-09-12. Google ships Gemini revisions frequently — re-verify this section before trusting a specific model name or parameter.

---

## Model landscape (verified September 2026)

| Model | API ID | Positioning |
|---|---|---|
| Gemini 3.8 Flash | `gemini-3.8-flash` | Latest and most capable Flash model — long-horizon software engineering, autonomous agents, complex enterprise workflows |
| Gemini 3.7 Flash | `gemini-3.7-flash` | Complex coding and agentic workflows |
| Gemini 3.6 Flash | `gemini-3.6-flash` | General tasks, balances speed and multimodal capability |
| Gemini 3.5 Flash | `gemini-3.5-flash` | Routine, high-throughput workloads |
| Gemini 3.5 Flash-Lite | `gemini-3.5-flash-lite` | Fast, cost-effective execution; high-volume subagents |
| Gemini 3.1 Pro | `gemini-3.1-pro-preview` | Preview — advanced problem-solving and agentic capabilities |

Gemini 2.0 models are shut down; Gemini 2.5 (`gemini-2.5-pro`, `gemini-2.5-flash`, `gemini-2.5-flash-lite`) is the previous generation. Gemini 3.8 Flash: 1M token context, 64K max output.

Specialized models exist for transcription (`gemini-3.5-transcribe`), image generation (Nano Banana family), video (Veo 3.1), music (Lyria 3.5), and embeddings — consult the live docs when a prompt targets one of these rather than assuming the text-model conventions apply.

---

## Generation config — what changed in Gemini 3+

This is the most common source of stale Gemini prompts and integration code:

| Parameter | Status on Gemini 3+ |
|---|---|
| `temperature`, `top_p`, `top_k` | **Remove them.** Google's migration guidance instructs developers to drop these from generation configs on Gemini 3+. Advice to "keep temperature at 1.0" is obsolete — the parameter should be absent, not set. |
| `candidate_count` | Unsupported on Gemini 3+ — remove. |
| `thinking_budget` | Deprecated — replaced by the `thinking_level` string enum. |
| `thinking_level` | `low` / `medium` (default) / `high`. `minimal` is **not** supported and returns an error. |

**`thinking_level` guidance:**
- `low` — latency-critical work: real-time chat, drafts, fast data analysis, incident response pipelines.
- `medium` (default) — best quality for most tasks; recommended for complex code and agentic use cases.
- `high` — deep reasoning, mathematics, difficult multi-step tasks, heavy tool orchestration.

Depth is now controlled by **configuration, not prose**. When a prompt targets a thinking-capable Gemini model, prefer raising `thinking_level` over adding "think carefully step by step" instructions to the prompt text.

---

## The PTCF Framework — Google's recommended structure

Every Gemini prompt should follow the **Persona · Task · Context · Format** order.

### Template

```
[PERSONA]
You are an expert in [domain] with 10 years of experience.

[TASK]
Your mission: [precise action + action verb]

[CONTEXT]
Here is the available information: [data, documents, URLs]

[FORMAT]
Present your response as a [table / list / JSON / paragraph] with [length / specific constraints].
```

---

## Pattern 1 — Standard Gemini task prompt (API system prompt)

```xml
<role>
You are a [specific role] with deep expertise in [domain].
</role>

<task>
[Precise description of what the model must accomplish. Use a clear action verb.]
</task>

<context>
You are working with [audience description]. They expect [tone/style].
The output will be used for [purpose].

[Additional context, data, or background information]
</context>

<constraints>
- Use only the information provided in context for your reasoning.
- Keep responses under [X] words.
- [Additional behavioral boundaries — stated positively]
</constraints>

<format>
[Description of expected output structure: JSON schema, table format, markdown template, etc.]
</format>

<examples>
<example>
Input: [Example user input]
Output: [Example ideal output]
</example>

<example>
Input: [Second example — different scenario]
Output: [Second example output]
</example>

<example>
Input: [Third example — edge case]
Output: [Third example output]
</example>
</examples>
```

---

## Pattern 2 — Analytical / reasoning prompt (with planning)

For Gemini, do NOT use `<thinking>` / `<answer>` tags.

**On thinking-capable models (Gemini 3+), reach for `thinking_level` first.** Set `thinking_level: "high"` and state the task and its verification criteria plainly — the model plans internally. Prompted planning scripts duplicate work the model already does and can make output worse.

Use the explicit planning template below when the prompt targets a non-thinking model, when the user needs the plan itself to appear in the output (an auditable deliverable, not a reasoning aid), or when `thinking_level` is not available in the destination surface.

```xml
<role>
You are a [role] specializing in [domain].
</role>

<task>
[The analytical task to perform]
</task>

<context>
[Background information relevant to the task]
</context>

<instructions>
Before providing your final answer, follow this process:

1. Break down the stated objective into distinct sub-tasks.
2. Verify whether the input information is complete.
3. Create a structured plan to achieve the objective.
4. Execute each step, noting your reasoning.
5. Review your output against the original task.
6. Present the final answer in the format specified below.
</instructions>

<format>
[Specify the structure of the final answer]
</format>

<examples>
<example>
Input: [Example input]
Plan: [Brief plan demonstration]
Answer: [Final structured answer]
</example>
</examples>
```

---

## Pattern 3 — Document processing prompt (long context)

Gemini supports up to 1M+ tokens in context. Place documents at the top.

```xml
<document>
{{DOCUMENT_CONTENT}}
</document>

<role>
You are a [role] tasked with [specific document task].
</role>

<task>
Using the document provided above, [precise extraction/analysis task].
</task>

<instructions>
1. [Primary extraction/analysis task]
2. [Secondary task if applicable]
3. Review your output for completeness and accuracy.

Use only the information from the provided document. If information is missing or ambiguous, note this explicitly rather than filling gaps with external knowledge.
</instructions>

<format>
[Specify structure: summary format, extraction schema, analysis template]
</format>
```

---

## Pattern 4 — Conversational assistant (AI Studio / chat)

Lighter structure, more natural tone. Gemini 3 responds well to concise instructions.

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

When you're unsure, say so rather than guessing. Base your reasoning on the information provided.

[1-2 short examples if the task has a specific format]
```

---

## Pattern 5 — Multimodal prompt (image / video / audio)

```xml
<role>
You are a [role] skilled at analyzing [media type].
</role>

<task>
Examine the provided [image/video/audio] and [specific task: identify, describe, extract, compare...].
</task>

<instructions>
- Focus on [specific elements to analyze].
- [Specific question: "What is the trend shown by the blue line?" rather than "Analyze this."]
- [Additional targeted questions if needed]
</instructions>

<format>
Present your findings as [JSON / bullet list / structured description].
[If applicable: "List all text labels from the diagram as a JSON array."]
</format>
```

**Key rules for multimodal prompts:**
- Refer to media naturally: "the image", "this chart" — no need for labels with a single media.
- Ask specific questions, not "analyze this."
- Always request a clear output format.

---

## Pattern 6 — Structured JSON output

When guaranteed JSON compliance is needed via the API, note the `responseSchema` parameter.

```xml
<role>
You are a data extraction specialist.
</role>

<task>
Extract structured data from the provided input and return it as valid JSON.
</task>

<instructions>
Parse the input and extract the following fields:
- [field_1]: [description and expected type]
- [field_2]: [description and expected type]
- [field_3]: [description — use null if not found]

Return only valid JSON matching the schema below. Do not include any text outside the JSON block.
</instructions>

<format>
```json
{
  "field_1": "string",
  "field_2": number,
  "field_3": "string | null"
}
```

Note: For API integration, use the `responseSchema` parameter to guarantee JSON compliance. Use nullable fields for optional data to avoid response refusal.
</format>

<examples>
<example>
Input: [Raw text example]
Output:
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

## Gemini-Specific Best Practices Summary

| Practice | Details |
|---|---|
| **PTCF framework** | Always use Persona · Task · Context · Format order |
| **Few-shot first** | Google recommends examples as the primary quality lever — include 3-5 |
| **Concise instructions** | Gemini 3 follows instructions well — don't over-specify |
| **Positive constraints** | Avoid broad negatives ("don't guess"). Say "use only the provided context" |
| **Sampling parameters** | Remove `temperature`, `top_p`, `top_k`, `candidate_count` on Gemini 3+ — they are not to be set |
| **Reasoning depth** | Control with `thinking_level` (`low`/`medium`/`high`), not with prose. `minimal` errors |
| **Consistent delimiters** | Pick XML *or* Markdown headers and stay with it — mixing formats blurs section boundaries |
| **Partial completion** | Start a structure (e.g., "I. Introduction\n*") and Gemini will continue the pattern |
| **Multimodal** | Ask specific questions about media, request clear output formats |
| **Long context** | Place documents at top, instructions below. Ask targeted questions |
| **JSON outputs** | Describe schema explicitly. Mention `responseSchema` for API use. Use nullable fields |

---

## Pitfalls to Avoid with Gemini

1. **Over-specifying for Gemini 3** — Prompts that were necessary for Gemini 2.x may be verbose overkill for Gemini 3. Start simple, add detail only if quality drops.
2. **Broad negative instructions** — "Never infer", "Don't guess" can cause Gemini to over-index and fail basic reasoning. Reframe positively.
3. **Missing few-shot examples** — Google explicitly states that prompts without examples tend to be less effective. Always include them when format matters.
4. **Setting sampling parameters at all** — `temperature`, `top_p`, `top_k` and `candidate_count` should be *absent* from Gemini 3+ generation configs, not tuned. Carrying them over from a Gemini 2.x integration is the most common migration defect.
5. **Using `thinking_budget` or `minimal` thinking** — `thinking_budget` is deprecated in favour of `thinking_level`, and `thinking_level: "minimal"` returns an error. Use `low` as the floor.
6. **Prompted chain-of-thought on thinking models** — "think step by step" duplicates internal reasoning. Raise `thinking_level` instead.
7. **Vague multimodal prompts** — "Analyze this image" produces poor results. Be specific about what to look for.
8. **Mixing XML and Markdown structure in one prompt** — pick one delimiter style so section boundaries stay unambiguous.

---

## XML Tag Conventions for Gemini

| Tag | Purpose |
|---|---|
| `<role>` | Who the model is (persona) |
| `<task>` | What the model must accomplish |
| `<context>` | Background info, data, documents |
| `<constraints>` | Boundaries, stated positively |
| `<instructions>` | Step-by-step process (for complex tasks) |
| `<format>` | Expected output structure |
| `<examples>` | Input/output demonstrations |
| `<document>` | Long-form content to process |
