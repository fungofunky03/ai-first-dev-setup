# Contributing to ai-first-dev-setup

Thanks for considering a contribution. This is a curated library, not an
open dumping ground — the value is in the quality bar, not the size of
the catalogue. Read this whole file before opening a PR for new content.

## What we accept

- **New prompts** that fill a real gap, follow the schema, and ship with a
  concrete example the author has run against a current frontier model.
- **New workflows** that capture a complete loop (not a single tip) and
  reference existing prompts.
- **Fixes** to prompts that don't perform well, with before/after evidence.
- **Translations** of `docs/philosophy.es.md` style — only when a
  Spanish speaker has audited the existing translation.
- **Customisation guides** for stacks not yet covered in
  `docs/customization.md`.

## What we don't accept

- Prompts copied from other repositories without attribution and without
  measurable improvement over the source.
- Prompts that only work against a single model, unless that model's
  unique feature is the whole point (and `tested_with` reflects this).
- Generic "make my code better" prompts. Every prompt must have a
  specific trigger condition.
- Marketing-flavoured READMEs, descriptions, or examples ("blazing fast",
  "robust", etc.).
- New top-level directories or filename conventions. Open an issue first
  to discuss structural changes.

## Before you open a PR

1. **Open an issue** describing what you plan to add and why. For small
   fixes, skip this step.
2. **Read the philosophy** (`docs/philosophy.md`). Make sure your
   contribution aligns. If it deliberately challenges a stated belief,
   say so in the PR description.
3. **Run your prompt at least once** against a current model. Copy the
   real output into the PR description as evidence.

## Prompt submission template

Every new prompt file under `prompts/<category>/<slug>.md` must pass this
checklist before review.

### Frontmatter

```yaml
---
title: "<Short, human-readable title>"
category: code-review | refactor | debug | docs | tests
when_to_use: "<One sentence on the trigger condition>"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---
```

Checklist:
- [ ] `title` is short (≤ 40 chars), human-readable, no marketing words.
- [ ] `category` is one of the five existing categories. If you think
      a new category is justified, open an issue first.
- [ ] `when_to_use` describes the trigger, not the goal. Bad: "improve
      your tests". Good: "before refactoring a function with thin test
      coverage".
- [ ] `tested_with` lists every model you actually ran the prompt
      against. Do not list models you did not test on.
- [ ] `version: 1.0` for new prompts. Bumps in subsequent PRs follow
      SemVer (`1.1` for non-breaking improvement, `2.0` for breaking
      output format changes).

### Body sections (in order)

1. **Description** — 1-2 paragraphs. Why this prompt exists; what
   failure mode it prevents; what makes it work.
2. **Prompt** — full prompt text in a fenced code block. This is the
   part users copy. Use placeholders like `<TARGET>`, `<FILE>`.
3. **Example** — a concrete before/after. Default stack is
   TypeScript / React; deviate only if the prompt is genuinely
   stack-agnostic (e.g. `changelog-from-commits.md`).
4. **Personalisation tips** — variables users should swap to fit their
   stack or team. At least 3 distinct tips.

Checklist:
- [ ] Description explains *why*, not just *what*. "This prompt is for
      X" is not enough.
- [ ] Prompt is self-contained: a user pasting it into a chat without
      reading the description should still get useful output.
- [ ] Example is real: you ran it and the output is paraphrased from
      actual model output, not invented.
- [ ] Personalisation tips cover at least: different stack, different
      audience seniority, and one edge case.

### Output check

- [ ] No marketing language ("blazing", "robust", "powerful", etc.).
- [ ] No emoji in file content unless the existing file already uses
      them.
- [ ] No second-person ("you will love this!") in descriptions.
- [ ] File is < 250 lines. Longer prompts indicate either bloat or a
      missing split point.

## Workflow submission template

Workflows are larger artefacts. Each new file under `workflows/`:

- [ ] States **when to apply** and **when NOT to apply**.
- [ ] Lists **required tools** (rules files, MCPs, IDE features).
- [ ] Has a **mermaid diagram** if the flow has more than 3 sequential
      steps.
- [ ] Has step-by-step instructions with explicit **expected output of
      each step**.
- [ ] Has an **anti-patterns** section listing common failure modes.
- [ ] References existing prompts where applicable (use full paths).
- [ ] Has been used by the author on a real project at least once.

## Style guide

- Tone: developer-native, direct, no marketing fluff.
- Voice: prefer "you" for instructions, third person for prose.
- Hedging: minimise. "Sometimes" and "often" are signals that the
  author hasn't decided. Be specific.
- Cite file paths in backticks: `src/index.ts`. Cite line numbers when
  relevant: `src/index.ts:42`.
- Prefer tables for multi-column comparisons; prefer bullets for lists
  with 3+ items; prefer prose for everything else.
- Anglo-style sentence-case headings.
- Line length: soft 80 chars in markdown, hard 100. Wrap on word
  boundaries.

## Review process

1. A maintainer triages within ~1 week.
2. We may request specific changes; the bar is high precisely because
   the library is curated.
3. Merges are squashed. The PR title becomes the commit message — write
   it in the imperative ("Add unit-test prompt for property-based
   testing").
4. Released artefacts get a `CHANGELOG.md` entry in the next version
   bump (Keep a Changelog format).

## Code of conduct

Be specific, be technical, be kind. The audience is professionals. The
repo's tone reflects that — assume good faith and contribute the same.

If you want to flag conduct issues privately, contact the maintainers
listed in the Learning-Heroes / HeroLabs GitHub org.
