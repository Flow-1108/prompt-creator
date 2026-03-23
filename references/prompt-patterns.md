# Prompt Patterns Reference

This file contains reusable structural patterns for generating high-quality Claude prompts. Referenced by the prompt-creator skill during generation.

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

## Pattern 2 — Analytical / reasoning prompt (with chain of thought)

Use this when the task involves analysis, classification, decision-making, or multi-step reasoning.

```xml
<role>
You are a [role] specializing in [domain].
</role>

<context>
[Background information relevant to the task]
</context>

<instructions>
For each request, follow this process:

1. Read the input carefully.
2. In a <thinking> block, work through your reasoning step by step:
   - Identify the key elements of the input.
   - Consider relevant factors or criteria.
   - Weigh alternatives if applicable.
   - Arrive at your conclusion.
3. In an <answer> block, provide your final response in the format specified below.

Your thinking should be thorough but efficient. Focus on the reasoning that actually influences your conclusion.
</instructions>

<examples>
<example>
<input>[Example input]</input>
<thinking>[Step-by-step reasoning demonstration]</thinking>
<answer>[Final structured answer]</answer>
</example>
</examples>

<output_format>
Always respond with a <thinking> block followed by an <answer> block.
[Specify the format of the answer block]
</output_format>
```

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
| `<thinking>` | Chain-of-thought reasoning block |
| `<answer>` | Final answer after reasoning |
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
