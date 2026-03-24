# Gemini Prompt Engineering Guide

Reference file for the prompt-creator skill. Contains Gemini-specific patterns, best practices, and structural templates based on official Google documentation (ai.google.dev, Vertex AI, March 2026).

---

## The PTCF Framework — Google's Official Structure

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

For Gemini, do NOT use `<thinking>` / `<answer>` tags. Instead, use explicit planning instructions.

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
| **Temperature** | Keep at default 1.0 for Gemini 3 — don't adjust |
| **Planning over thinking tags** | Use explicit step-by-step planning, not `<thinking>` blocks |
| **Partial completion** | Start a structure (e.g., "I. Introduction\n*") and Gemini will continue the pattern |
| **Multimodal** | Ask specific questions about media, request clear output formats |
| **Long context** | Place documents at top, instructions below. Ask targeted questions |
| **JSON outputs** | Describe schema explicitly. Mention `responseSchema` for API use. Use nullable fields |

---

## Pitfalls to Avoid with Gemini

1. **Over-specifying for Gemini 3** — Prompts that were necessary for Gemini 2.x may be verbose overkill for Gemini 3. Start simple, add detail only if quality drops.
2. **Broad negative instructions** — "Never infer", "Don't guess" can cause Gemini to over-index and fail basic reasoning. Reframe positively.
3. **Missing few-shot examples** — Google explicitly states that prompts without examples tend to be less effective. Always include them when format matters.
4. **Adjusting temperature** — Gemini 3's reasoning is calibrated for temperature 1.0. Lowering it can degrade output quality.
5. **Vague multimodal prompts** — "Analyze this image" produces poor results. Be specific about what to look for.

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
