# GPT / OpenAI Prompt Engineering Guide

Reference file for the prompt-creator skill. Contains GPT-specific patterns, best practices, and structural templates based on official OpenAI documentation (cookbook.openai.com, OpenAI API docs, March 2026).

---

## The CTCO Framework — OpenAI's Official Structure

Every GPT prompt should follow the **Context → Task → Constraints → Output** order. Separating constraints from the task reduces instruction drift in long context windows.

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

## Pattern 2 — Reasoning prompt (standard GPT models)

For standard GPT models (not reasoning/thinking models), ask for a brief reasoning preamble.

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

**Important:** For reasoning/thinking models (o3, GPT-5.x Thinking), do NOT include reasoning instructions. These models have internal chain of thought. Keep the prompt simpler:

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

## Pattern 3 — Agent prompt (with persistence + planning + tools)

The 3 key instruction categories that improved SWE-bench scores by ~20%.

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

<planning>
Before taking action:
1. Analyze the task and identify sub-steps.
2. Create a brief plan.
3. Execute each step, tracking progress.
4. After completion, verify your work against the original request.
</planning>

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
| **3 agent instructions** | Persistence + Planning + Tool usage for agent prompts |
| **Reasoning preamble** | Ask standard GPT models to summarize reasoning as bullet points |
| **No CoT for thinking models** | o3 / GPT-5.x Thinking already reason internally — keep prompts simple |
| **Reasoning effort** | API parameter: "low" for extraction, "medium" baseline, "high" for complex analysis |
| **Pair negatives with positives** | "Don't X" must be followed by "Do Y instead" |
| **Few-shot examples** | 3-5 examples. Must be consistent with instructions (no contradictions) |
| **Structured Outputs** | `response_format` with JSON schema constrains at token level |
| **Persona adoption** | GPT responds very well to strong persona definitions |

---

## Pitfalls to Avoid with GPT

1. **Adding "think step by step" to reasoning models** — o3 and GPT-5.x Thinking already have internal CoT. Adding explicit reasoning instructions is redundant and can degrade output quality.
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
| `<planning>` | Agent: think before acting |
| `<tool_usage>` | Agent: how to use available tools |
| `<instructions_top>` | Long context: instructions before document |
| `<instructions_bottom>` | Long context: instructions after document |
| `<document>` | Long-form content to process |
