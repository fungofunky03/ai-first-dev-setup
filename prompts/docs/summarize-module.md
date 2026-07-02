---
title: "Guided, layered summary of a code module"
category: docs
when_to_use: "When a module is large or unfamiliar and you need a summary at three altitudes — one line, one paragraph, and a structured reference — rather than a single flat wall of prose"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

A raw "summarise this file" prompt produces mush: it flattens a 400-line module
into one paragraph that is too vague to act on and too long to skim. Anthropic's
[`capabilities/summarization/`](https://github.com/anthropics/claude-cookbooks/tree/main/capabilities/summarization)
cookbook shows the fix — *guided summarization*: give the model the exact
structure you want and the domain lens to summarise through, instead of leaving
both to chance.

This prompt applies that to source code. It asks for the summary at three
altitudes — a one-liner for a directory index, a paragraph for a PR
description, and a structured reference for onboarding — because the right
length depends on where the summary will live. The guided template also forces
the model to name the module's *public contract* and *non-obvious behaviours*
separately, which is exactly the information a flat summary drops first.

## Prompt

```
Summarise the module <MODULE_PATH> for a developer who has never opened it.
Read the code; do not guess from the filename. Produce the summary through a
"what would a new maintainer need" lens — favour contracts and surprises over
line-by-line narration.

Output these layers, in order:

### One-liner
A single sentence (<= 20 words) suitable for a directory index or file
header. State what the module is responsible for, not how it works.

### Paragraph
3–5 sentences suitable for a PR description or a Slack answer to "what does
this do?". Cover the primary responsibility, its main collaborators
(what it imports/calls and what calls it, if inferable), and the one thing
a reader is most likely to get wrong about it.

### Reference
A structured breakdown:
- **Public surface:** every exported symbol with a one-line description and
  its `path:line`. This is the contract other code depends on.
- **Key internal pieces:** private helpers or state that a maintainer must
  understand to change the module safely. Skip trivial one-liners.
- **Non-obvious behaviour:** side effects, ordering constraints, implicit
  invariants, error/edge handling, anything surprising. Cite `path:line`.
  This is the section that earns the summary its keep.
- **Dependencies & blast radius:** what this module depends on and, as far
  as the code shows, what would break if its public surface changed.

Constraints:
- Cite `path:line` for every claim in the Reference layer.
- Do NOT restate the code in English line by line. Summarise behaviour.
- If something is genuinely unclear from the code alone (e.g. why a magic
  number exists), say "unclear from source" rather than inventing a reason.
- Do not propose changes. This is documentation, not review.

Module:
<paste the file(s)>
```

## Example

**Input (TypeScript):** `src/lib/rateLimiter.ts` — a token-bucket limiter with
a sliding refill and a per-key map.

**Expected output (excerpt):**

> ### One-liner
> Per-key token-bucket rate limiter with lazy, time-based refill.
>
> ### Paragraph
> `rateLimiter.ts` decides whether a given key (user id, IP) may proceed,
> tracking a token bucket per key in an in-memory `Map`. Buckets refill lazily
> on read based on elapsed time rather than on a timer, so an idle key costs
> nothing. The thing readers get wrong: it is **per-process** — behind more
> than one instance you are enforcing N× your configured limit, so it is only
> correct for single-process deploys or must be backed by shared state.
>
> ### Reference
> **Public surface**
> - `createRateLimiter(opts)` — `rateLimiter.ts:12` — factory; returns a
>   `{ tryConsume(key) }` object.
> - `tryConsume(key: string): boolean` — `rateLimiter.ts:20` — returns `true`
>   and decrements a token, or `false` if the bucket is empty.
>
> **Key internal pieces**
> - `buckets: Map<string, Bucket>` — `rateLimiter.ts:9` — unbounded; see
>   blast radius.
> - `refill(bucket, now)` — `rateLimiter.ts:31` — clamps refilled tokens to
>   `capacity`; the sliding-window logic lives here.
>
> **Non-obvious behaviour**
> - `rateLimiter.ts:9` — the `buckets` Map is **never evicted**. A high-
>   cardinality key space (per-request IDs) leaks memory unbounded.
> - `rateLimiter.ts:34` — refill uses `Date.now()`; a backward clock jump
>   (NTP correction) can stall refills. Unclear from source whether this was
>   considered.
>
> **Dependencies & blast radius**
> - No external deps. Consumed by `middleware/throttle.ts:8`. Changing the
>   `tryConsume` signature breaks that middleware and its tests.

## Personalisation tips

- **Destination-specific:** if the summary is going into a specific place,
  keep only that layer. Append "Output only the One-liner" for a directory
  README, or "Output only the Reference" for an onboarding doc.
- **Long modules / chunking:** for files beyond a model's comfortable context,
  summarise each exported symbol's neighbourhood separately, then feed those
  summaries back in and ask for the One-liner and Paragraph — the map-reduce
  approach from the cookbook. `changelog-from-commits.md` uses the same
  reduce step.
- **Architecture docs:** run this across every module in a package, then feed
  the collected One-liners into `readme-generator.md` to draft a package
  overview from the parts.
- **Domain lens:** swap "a new maintainer" for a role that matches the reader —
  "a security reviewer" surfaces trust boundaries and input handling; "an SRE"
  surfaces failure modes and observability gaps.
