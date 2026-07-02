---
title: "Build an LLM-as-judge eval for a non-deterministic feature"
category: tests
when_to_use: "When the thing you need to test returns free-form text (an LLM call, a summariser, a generated message) and a strict equality assertion cannot express 'good enough'"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Some outputs cannot be asserted with `toBe`. When a function calls a model to
summarise a ticket, draft a reply, or extract intent, two correct runs will
differ word-for-word. The industry answer, from Anthropic's
[`misc/building_evals.ipynb`](https://github.com/anthropics/claude-cookbooks/blob/main/misc/building_evals.ipynb)
cookbook, is a *model-graded* eval: a second model scores the output against a
rubric you define, and you assert on the score.

This prompt builds that harness for you. It refuses to hand-wave the rubric —
it forces you to name the *failure modes you actually care about* and grade
each independently, because a single 1–10 "quality" score is noise. The
non-obvious move borrowed from the cookbook is separating the **grader prompt**
(reusable, deterministic-ish, low temperature) from the **eval set** (the
inputs + expected properties), so you can grow the set without touching the
grader.

## Prompt

```
I need an LLM-as-judge evaluation for <FEATURE>, which takes <INPUT_SHAPE>
and returns free-form text: <OUTPUT_SHAPE>. I cannot assert exact equality.

Follow the model-graded eval pattern (rubric-based scoring by a second
model). Produce four artefacts, in this order:

1. **Failure-mode inventory.** Before any code, list the 3–6 specific ways
   this output goes wrong in production (e.g. "hallucinates a policy that
   was not in the source", "leaks the customer's email", "answers a
   different question than asked"). These become the graded dimensions.
   Do NOT propose a generic "overall quality" score.

2. **Grader prompt.** A self-contained prompt for the judge model. It must:
   - receive the input, the output-under-test, and (if available) a
     reference answer;
   - score EACH failure mode from (1) as pass/fail with a one-line reason,
     never a vague number;
   - return machine-parseable output (strict JSON: one boolean per
     dimension + a single `reasons` object). Specify the exact schema.
   - be told to grade ONLY what is present, never to rewrite or improve the
     output, and to default to "fail" when uncertain.

3. **Eval set.** 6–10 input cases as data, not prose. Cover: a clean
   happy path, at least two adversarial inputs designed to trigger a
   failure mode from (1), one empty/degenerate input, and one input where
   the *correct* behaviour is to refuse or ask for clarification. For each
   case, state which dimensions must pass.

4. **Runner.** A small harness (in <LANGUAGE/TEST_FRAMEWORK>) that runs the
   feature over the eval set, calls the grader on each output at low
   temperature, and asserts the pass-rate per dimension against a
   threshold I can configure. Make the threshold a constant with a comment
   explaining why it is not 100% (judges are noisy).

Constraints:
- Grade dimensions independently. A hallucination and a tone problem are
  not the same failure and must not average together.
- The grader must be cheaper/faster to run than the feature is expensive to
  get wrong — note if a smaller model is appropriate for grading.
- Flag any dimension that a deterministic check (regex, schema validation,
  string-contains) could grade instead. Prefer the cheap check; only send
  to the judge what genuinely needs judgement.

Feature under test:
<paste the function / prompt / endpoint>
```

## Example

**Input (TypeScript):** a helper that summarises a support ticket for an
agent's queue.

```ts
// src/support/summariseTicket.ts
export async function summariseTicket(ticket: {
  subject: string;
  body: string;
  customerEmail: string;
}): Promise<string> {
  const res = await claude.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 120,
    messages: [
      {
        role: "user",
        content: `Summarise this ticket in 2 sentences for an agent:\n\n${ticket.subject}\n${ticket.body}`,
      },
    ],
  });
  return res.content[0].text;
}
```

**Expected output (excerpt):**

> ### 1. Failure-mode inventory
> - **F1 — PII leak:** the summary includes `customerEmail`, which the agent
>   already has and which should never be duplicated into queue previews.
> - **F2 — Fabrication:** the summary states a fact (an order number, a
>   promised refund) not present in `subject` or `body`.
> - **F3 — Length blowout:** more than 2 sentences, defeating the point of a
>   queue preview.
> - **F4 — Wrong subject:** summarises a different concern than the ticket's
>   primary ask.
>
> ### 2. Grader prompt
> ```
> You are grading a support-ticket summary. You are given the ORIGINAL
> ticket and the SUMMARY. Grade only what is present. Do not rewrite.
> Return strict JSON: {"f1_no_pii": bool, "f2_grounded": bool,
> "f3_length_ok": bool, "f4_on_topic": bool, "reasons": {...}}.
> - f1_no_pii: true unless the summary contains an email address.
> - f2_grounded: true only if every claim appears in the ORIGINAL. If
>   uncertain whether a claim is supported, return false.
> ...
> ```
>
> ### 3. Eval set
> ```ts
> const CASES = [
>   { name: "happy path", ticket: {...}, mustPass: ["f2","f3","f4"] },
>   { name: "adversarial: body invites fabrication ('what's my refund?')",
>     ticket: {...}, mustPass: ["f2"] },
>   { name: "PII bait: email repeated in body", ticket: {...}, mustPass: ["f1"] },
>   ...
> ];
> ```
>
> ### 4. Runner
> ```ts
> const THRESHOLD = 0.9; // judges are noisy; require 90% not 100%
> // ...for each dimension, assert passRate >= THRESHOLD
> ```
>
> Note: **F1 and F3 do not need the judge.** `summary.includes("@")` and a
> sentence count grade them deterministically for free. Send only F2 and F4
> to the model.

## Personalisation tips

- **Cheaper grading:** for high-volume evals, grade with a smaller/faster
  model than the feature uses. Note that a small judge is weakest on F2-style
  grounding checks — keep those on the stronger model.
- **Golden references:** if you have human-written "ideal" outputs, add them
  to each case and instruct the grader to score *relative to* the reference.
  This tightens F4-style judgements considerably.
- **CI integration:** model-graded evals are non-deterministic and cost
  tokens. Run them on a nightly job or a `[eval]` label, not on every push —
  see the `pr-workflow.md` workflow for where this fits.
- **Regression tracking:** persist per-dimension pass-rates over time. A drop
  in one dimension after a prompt change is the signal; the aggregate score
  hides it. Pairs well with `edge-case-finder.md` for finding the adversarial
  inputs worth adding to the eval set.
