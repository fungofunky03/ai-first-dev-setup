---
title: "Triage a bug report or stack trace into a routed classification"
category: debug
when_to_use: "When an incoming bug report, exception, or alert needs to be classified and routed — not yet fixed — and you want a consistent, machine-parseable triage instead of an ad-hoc guess"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Triage is a classification problem, and Anthropic's
[`capabilities/classification/`](https://github.com/anthropics/claude-cookbooks/tree/main/capabilities/classification)
cookbook is explicit about what makes classification reliable: a *fixed label
set* the model must choose from, a *forced structured output*, and a
*calibrated confidence* with an escalation path when the model is unsure. A
free-text "this looks like a caching issue, probably?" is none of those and
cannot be routed automatically.

This prompt turns a raw report or stack trace into a triage record: component,
category, severity, a confidence score, and a routing decision — using label
sets *you* define so the output maps onto your actual teams and dashboards. It
deliberately does not attempt a fix: triage that also speculates on the fix
tends to anchor on the first plausible cause and mislabel severity to match.
Once routed, hand off to `investigate-bug.md` or `root-cause-analysis.md`.

## Prompt

```
Triage the following bug report / stack trace. Classify only — do NOT
propose a fix or root cause yet.

Choose every label from these fixed sets. If nothing fits, use "other" and
say why; never invent a label.

- component: <e.g. auth | billing | api | web-ui | worker | infra | other>
- category: <regression | crash | data-corruption | perf | ux |
  security | config | external-dependency | other>
- severity: sev1 (data loss / outage / security) | sev2 (major feature
  broken, no workaround) | sev3 (broken with workaround) | sev4 (minor / cosmetic)

Return STRICT JSON with exactly these fields:
{
  "component": "<label>",
  "category": "<label>",
  "severity": "<label>",
  "confidence": 0.0-1.0,      // your calibrated confidence in the above
  "signals": ["<the specific tokens from the input that drove each label>"],
  "duplicate_of": "<issue ref or null> — only if the input itself names one",
  "route_to": "<team/queue implied by component+severity>",
  "needs_human": <true if confidence < 0.6 OR severity is sev1>,
  "missing_info": ["<what you'd need to raise confidence, if anything>"]
}

Rules:
- Base labels ONLY on evidence in the input. Put the evidence in `signals`.
  If a field is a guess, lower `confidence` accordingly — do not round up.
- A stack trace's top non-framework frame usually indicates `component`; say
  so in `signals` when you use it.
- Default `severity` UP when the report mentions data, money, auth, or
  "everyone/all users"; default DOWN for single-user cosmetic issues.
- Set `needs_human: true` and stop if the input is too sparse to classify —
  do not pad the JSON with low-confidence guesses.

Report:
<paste report, exception, or alert>
```

## Example

**Input:** an unhandled exception from production logs.

```
TypeError: Cannot read properties of undefined (reading 'currency')
    at formatTotal (src/billing/invoice.ts:88:31)
    at renderInvoice (src/billing/invoice.ts:142:20)
    at InvoicePage (src/pages/invoice.tsx:24:15)
  context: 3 occurrences in 10 min, all for accounts created after the
  2026-06-30 pricing migration
```

**Expected output:**

```json
{
  "component": "billing",
  "category": "regression",
  "severity": "sev2",
  "confidence": 0.82,
  "signals": [
    "top app frame formatTotal at src/billing/invoice.ts:88 → component=billing",
    "'accounts created after the pricing migration' → regression tied to a dated change",
    "reading 'currency' of undefined → a field the migration likely stopped populating",
    "3 occurrences / 10 min, subset of accounts → broken feature, not full outage → sev2 not sev1"
  ],
  "duplicate_of": null,
  "route_to": "billing-team",
  "needs_human": false,
  "missing_info": [
    "whether the currency field is null in the DB or only undefined in the response shape",
    "total blast radius: how many post-migration accounts hit this path"
  ]
}
```

## Personalisation tips

- **Your label sets are the whole game:** replace the `component`,
  `category`, and `severity` values with the exact ones your issue tracker
  and on-call rotation use. The prompt is only as useful as these lists are
  faithful — a label the model can't map to a team is a triage that can't
  be routed.
- **Auto-route vs. human gate:** tune the `needs_human` threshold to your
  risk appetite. Teams new to auto-triage should raise it (e.g. `< 0.8`) and
  review the model's calls for a week before trusting the routing.
- **Duplicate detection:** this prompt only flags a duplicate the report
  itself names. To detect true duplicates, retrieve the top-k similar open
  issues first and paste their titles + IDs into the prompt so the model can
  match against real candidates.
- **Batch triage:** to classify a backlog, send reports in batches and ask
  for a JSON array. Keep the label sets in the system prompt so they are not
  repeated per item, and spot-check the low-confidence rows by hand.
- **Handoff:** once `route_to` is set and `needs_human` is false, feed the
  same report into `investigate-bug.md` to start the actual diagnosis — the
  triage record becomes that prompt's starting context.
