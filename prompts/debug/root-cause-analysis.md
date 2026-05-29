---
title: "Root cause analysis (post-incident)"
category: debug
when_to_use: "After an incident has been mitigated, when you need a blameless RCA that distinguishes the trigger from the underlying weakness"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Post-incident RCAs tend to stop at the trigger — "we deployed config X" —
without identifying the *system property* that made the trigger fatal.
This prompt enforces the distinction between trigger, contributing factors,
and root cause, and asks for prevention measures classified by where they
attack the failure (detection, blast radius, fix-forward speed).

It is deliberately blameless: the model is instructed not to attribute
fault to individuals, only to systems and processes.

## Prompt

```
You are writing a root cause analysis for an incident. The goal is to
identify what the system needs to change so a similar incident is either
prevented or detected faster, NOT to find who to blame.

Output exactly these sections:

### Incident summary
- Start time (UTC), end time, duration.
- User impact, in concrete terms (e.g. "30% of API requests returned 5xx
  for 17 minutes"). Avoid vague language like "degradation".

### Timeline
A bullet list with timestamps (UTC) of: trigger event, first detection
(automated or manual), mitigation start, mitigation complete, all-clear.

### Trigger
The single proximate event that started the incident. One sentence.

### Contributing factors
Conditions that turned the trigger into an incident. For each, note
whether removing it would have prevented the incident or merely shortened
it.

### Root cause
The underlying system property that allowed the trigger to cause harm.
Distinguish this from the trigger: the trigger is "what happened", the
root cause is "why the system was vulnerable to that thing happening".

### Why didn't we catch this earlier?
Was there a monitoring gap, a missed alert, an alert that fired but was
not actioned? Be honest about detection lag.

### Action items
Three categories. At least one item per category, unless that category is
genuinely empty (then say why).

  - **Prevention** — changes that stop this class of trigger from
    occurring again.
  - **Detection** — monitoring/alerting changes that surface the same
    failure mode within <N> minutes next time.
  - **Mitigation** — runbooks, automation, or capabilities that reduce
    user impact if it does recur.

Each action item: owner (role, not name), estimated effort (S/M/L),
priority (P0/P1/P2), and a one-line definition-of-done.

Constraints:
- Do not name individuals. Use roles ("on-call SRE", "release engineer").
- Do not list lessons-learned platitudes ("we should communicate better").
  Every lesson must convert into an action item.
- If a contributing factor is "we did not have <X>", explicitly state
  whether <X> should now exist (action item) or was deliberately not
  prioritised (no action item, with reasoning).

Incident details:
<paste relevant Slack threads, alerts, deploy logs, commits, dashboards>
```

## Example

**Input:** A condensed incident summary about a 17-minute API outage
caused by a config push that set a connection pool to 0.

**Expected output (excerpt):**

> ### Incident summary
> - Start: 2026-04-12 14:03 UTC. End: 14:20 UTC. Duration: 17 minutes.
> - Impact: 32% of `/api/*` requests returned `503 Service Unavailable`.
>   ~14k user-facing errors. No data loss.
>
> ### Trigger
> A config-only deploy at 14:03 set `DB_POOL_SIZE=0` in the
> `payments-api` Kubernetes deployment, exhausting database connections
> within seconds.
>
> ### Contributing factors
> - Config validation accepts any integer ≥ 0 for `DB_POOL_SIZE`. A pool
>   of 0 is syntactically valid but semantically broken. Removing this
>   would have prevented the incident.
> - Config-only deploys are not behind canary. Removing this would have
>   limited blast radius to ~5% of traffic for ~2 minutes, but not
>   prevented.
> - The `/healthz` endpoint does not require a successful DB query.
>   Healthcheck passed throughout the incident, so no automated rollback.
>   Removing this would have shortened the incident to ~5 minutes.
>
> ### Root cause
> Config schema does not encode operationally valid value ranges. The
> system trusts any value that parses, even when the value is incompatible
> with the workload.
>
> ### Why didn't we catch this earlier?
> The 503 surge alert fired at 14:05 (within SLO). Mitigation started at
> 14:11 because the on-call SRE first investigated a coincidental
> upstream alert. Improved alert correlation would shave ~6 minutes.
>
> ### Action items
>
> Prevention:
> - [P0, S] Add config schema constraint `DB_POOL_SIZE >= 5`. Owner:
>   platform engineer. DoD: a PR with `DB_POOL_SIZE=0` fails CI.
> - [P1, M] Canary config-only deploys (10% / 5 min / auto-rollback on
>   error rate). Owner: release engineer.
>
> Detection:
> - [P0, S] `/healthz` must require a successful DB ping. Owner:
>   payments-api maintainer. DoD: synthetic broken-DB test goes red.
>
> Mitigation:
> - [P1, S] Runbook entry: "On 503 surge after a config-only deploy, run
>   `kubectl rollout undo` first, investigate second." Owner: SRE lead.

## Personalisation tips

- **Different framework:** if your org uses 5-whys or fishbone diagrams,
  add "Format the Root Cause section as a 5-whys chain" or "Generate a
  fishbone diagram in mermaid for Contributing Factors".
- **Compliance-driven RCAs:** if you need a customer-facing version, add
  "Output a separate `customer_summary` section in plain non-technical
  language, ≤120 words, with no internal infrastructure names."
- **Sev-1 vs Sev-3:** for low-severity incidents, drop the "Why didn't
  we catch this earlier?" section to avoid forcing weak conclusions. The
  rest scales down naturally.
- **Pre-mortem:** invert the prompt for pre-mortems by replacing
  "incident" with "planned change" and "Trigger" with "Most likely
  failure mode". Useful before risky launches.
