# Philosophy

This repository is opinionated. Reading the rules, prompts, and workflows
without understanding why they exist will lead you to reject the ones
that disagree with your current habits — and those are often the ones
worth keeping.

This document explains the load-bearing beliefs behind everything in
`ai-first-dev-setup`. There is a Spanish translation at
[`philosophy.es.md`](./philosophy.es.md).

## The bet

AI-assisted development is now the default, not a novelty. Within a
working week most engineers will hit one of two outcomes:

- **They lean into it deliberately.** They configure their tools so the
  AI has accurate project context, they write prompts that get useful
  output on the first try, and they treat the AI as a fast collaborator
  with predictable failure modes.
- **They use it accidentally.** They paste code into a chat, accept
  whatever comes back, and ship it. They are slightly faster than they
  used to be and substantially less reliable than they should be.

This repo is for the first group. The artefacts here are the
configuration, vocabulary, and rituals that turn an AI assistant from
an autocomplete-on-steroids into a structured collaborator.

## Five beliefs

### 1. Context beats cleverness

The single biggest determinant of AI output quality is how much accurate
project context the model has. A correct, narrow prompt against a model
that knows your conventions beats a brilliantly crafted prompt against
a model that does not.

This is why most of this repo is configuration (`CLAUDE.md`,
`.cursorrules`, `agent.md`, MCP configs), not prompt templates. The
prompts assume the configuration is in place.

### 2. Plan, confirm, execute — every time

LLMs are at their best with a clear plan and at their worst when
inferring intent from vague requests. The "plan, confirm, execute"
ritual costs ~30 seconds and prevents the most expensive failure mode
in AI-assisted work: half-implemented changes you have to unwind.

Every workflow and prompt in this repo enforces this cadence. If a tool
or model tries to skip it, that is a signal to slow down, not to embrace
the speed.

### 3. The smallest reviewable unit

Code that gets reviewed in detail catches bugs; code that gets
rubber-stamped does not. Both AI and human reviewers rubber-stamp big
diffs. The fix is structural — produce work in small slices that
*demand* a real review at each step.

You will see this principle baked into prompts ("show me the diff after
each slice"), workflows ("pause between batches"), and even the
`.cursorrules` ("output incremental progress"). It is not optional.

### 4. The spec is the artefact

A common mistake with AI-assisted work is to iterate on the *code* when
the underlying confusion is in the *spec*. The code looks wrong because
the spec was wrong; rewriting the code without rewriting the spec just
moves the bug.

We bias toward spec-first work: write the smallest spec that captures
the change, critique it with the AI, then generate code from it. When
implementation reality forces a spec change, change the spec first as
a separate commit. The spec is the source of truth; the code is its
projection.

### 5. Verify behaviour, not output

The AI's job is done when the behaviour matches the requirement, not
when it says "Done." Tests passing is necessary but not sufficient.
For user-facing changes: click the button yourself. For backend
changes: hit the endpoint. For data changes: run a real query against
real data. The cost of this verification is small; the cost of skipping
it is borne by users.

## What we deliberately exclude

- **Prompts for "make my code better".** Vague intent produces vague
  output. Every prompt here has a specific trigger condition.
- **"AI personas"** ("You are a 10x engineer named Bob"). Theatre, not
  signal. Frontier models work better with direct, technical instructions
  than with persona priming.
- **Single-model assumptions.** Prompts are tested against multiple
  current models. If they only work on one, they are too brittle to
  ship.
- **Tooling beyond the assistant + IDE.** No custom CLIs, no orchestration
  frameworks. The tooling surface is what you already have; the leverage
  is in how you use it.

## How to disagree with this repo

If a rule, prompt, or workflow here does not fit your situation: change
it. Fork it. Open a PR with the change and the reason. This repo is
opinionated but not dogmatic — the audience is professionals making
context-specific decisions, not novices following a script.

The one ask: when you disagree, change the artefact in writing rather
than only in practice. Drift between what your config says and what you
actually do is worse than either extreme.
