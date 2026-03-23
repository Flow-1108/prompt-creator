# Prompt Creator — Claude Skill

A Claude Code skill that guides you through creating optimized prompts for any task.

## What it does

When triggered, this skill starts a natural conversation to collect context, then generates a single ready-to-use prompt — no explanations, no commentary, just the prompt.

It adapts the output based on the destination:
- **claude.ai conversation** — conversational, natural, moderate length
- **API system prompt** — structured, exhaustive, covers edge cases

## Trigger phrases

- "create a prompt"
- "write a system prompt"
- "improve my prompt"
- "generate instructions for Claude"
- "help me write a prompt for..."
- "I need a good prompt for..."

## Structure

```
├── SKILL.md                      # Main skill file
└── references/
    └── prompt-patterns.md        # Reusable prompt structures (5 patterns)
```

## Prompt engineering principles applied

- XML semantic tags (`<role>`, `<context>`, `<instructions>`, `<examples>`, `<output_format>`)
- Few-shot examples (3–5) for format/tone-sensitive tasks
- Chain-of-thought with `<thinking>` / `<answer>` for analytical tasks
- Long documents placed at the top of the prompt
- Calm, direct tone — no `ALL CAPS`, no `YOU MUST`, no `NEVER EVER`

## Installation

Place the skill folder inside your Claude Code plugins directory:

```
~/.claude/plugins/your-plugin/skills/prompt-creator/
```

## Usage

Works on both claude.ai and Claude Code.
