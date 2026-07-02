# Prompts library

A curated set of prompts for everyday engineering work, organised by intent.
Every prompt is production-ready: tested against current frontier models,
written in a consistent schema, and shipped with a concrete example you can
adapt.

## Categories

| Directory               | When to reach for it                                          |
| ----------------------- | -------------------------------------------------------------- |
| `code-review/`          | Reviewing a teammate's PR or your own pre-PR diff.            |
| `refactor/`             | Restructuring existing code without changing behaviour.       |
| `debug/`                | Investigating a failure mode you can't immediately explain.   |
| `docs/`                 | Generating or updating documentation from code or history.    |
| `tests/`                | Writing or extending automated tests.                         |

## File schema

Every prompt file follows the same shape.

### YAML frontmatter

```yaml
---
title: "Short, human-readable title"
category: code-review | refactor | debug | docs | tests
when_to_use: "One sentence on the trigger condition"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---
```

### Body sections

1. **Description** — 1-2 paragraphs on what the prompt is for and why it works.
2. **Prompt** — the full prompt text inside a fenced block. This is the part
   you copy into your IDE.
3. **Example** — a concrete before/after using a TypeScript/React snippet,
   unless the prompt is explicitly stack-agnostic (e.g. changelog generators).
4. **Personalisation tips** — variables you should swap to fit your stack
   or team conventions.

## Quick templates

If you want a bare fill-in-the-blank scaffold rather than a full, tested
prompt, see the
[prompt templates cheat sheet](../docs/prompt-templates-cheat-sheet.md).
It covers the same categories plus a few more (feature implementation,
performance, security, migrations) and links back to the full prompt in
this library wherever one exists.

## How to use a prompt

1. Open the prompt file and read the description to confirm it matches your
   situation. Many failures come from using a prompt for the wrong intent.
2. Copy the contents of the **Prompt** block.
3. Apply the personalisation tips before pasting.
4. Paste into your IDE chat. Attach the relevant file or diff as context.

## How to propose a new prompt

See `CONTRIBUTING.md` at the repo root. Short version: open a PR with a new
file in the right category directory, follow the schema, include a real
example you have personally tested against at least one frontier model.
