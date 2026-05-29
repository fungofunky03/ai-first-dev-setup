---
title: "Investigate a bug"
category: debug
when_to_use: "When something is broken in a way you can reproduce but cannot yet explain"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

A hypothesis-driven investigation prompt. The default failure mode of AI
assistants on bugs is to propose a fix immediately — usually a `try/catch`
that papers over the symptom. This prompt forces the model to enumerate
hypotheses, rank them, and verify before changing anything.

The structure mirrors how experienced engineers debug: state the
observable, generate hypotheses, find the cheapest way to falsify each,
then act on the survivor.

## Prompt

```
You are investigating a bug. Do not propose a fix yet. Do not write code
yet.

Output exactly these sections:

### 1. Observable
Restate the bug in one sentence: the trigger, the expected behaviour, the
actual behaviour. Cite the user's reproduction steps verbatim if provided.

### 2. Hypotheses
List 3-5 distinct hypotheses for the root cause, each in this form:
  - **H<N>:** one-sentence claim about the cause.
  - **Mechanism:** how this would produce the observable.
  - **Likelihood:** High / Medium / Low — based on the evidence so far.
  - **Cheapest falsifier:** the single fastest experiment (log line, unit
    test, command, query) that would disprove it.

Order by likelihood, highest first.

### 3. Recommended next experiment
Pick the highest-likelihood hypothesis that also has a cheap falsifier.
State exactly what to run, what successful falsification looks like, and
what failure to falsify means.

### 4. What I would need to see
Files, logs, stack traces, environment details that would let you narrow
the hypotheses further. Be specific (path, log query, env var name).

Constraints:
- Do not skip ahead to a fix.
- Do not say "could be many things" — commit to specific hypotheses.
- If the evidence is too thin for any hypothesis to reach Medium+, say so
  in Section 2 and only ask for more info in Section 4.

Bug report:
<paste reproduction, error message, stack trace, suspected files>
```

## Example

**Input:**

> Users sometimes get logged out mid-session, around the time they upload
> a file. Not always. We use NextAuth with a JWT session strategy. Recent
> change: we added Cloudflare R2 for direct uploads (presigned URLs).
> Stack trace at logout: `JWTError: signature verification failed`.

**Expected output (excerpt):**

> ### 1. Observable
> Users are intermittently logged out during sessions in which they
> upload files. Logout corresponds to a `JWTError: signature verification
> failed`. Started after introducing Cloudflare R2 presigned-URL uploads.
>
> ### 2. Hypotheses
>
> - **H1:** The `NEXTAUTH_SECRET` env var differs between two server
>   instances behind the load balancer. A request bounces to an instance
>   that cannot verify the cookie signed by another.
>   **Mechanism:** JWT signed with secret A, verified against secret B,
>   verification fails, session is dropped.
>   **Likelihood:** High — fits "intermittent", and recent infra change
>   likely introduced a second instance or env divergence.
>   **Cheapest falsifier:** `kubectl get pods -o yaml | grep NEXTAUTH_SECRET`
>   on each replica, compare values.
>
> - **H2:** The R2 upload SDK is writing to `document.cookie` or similar
>   and corrupting the session cookie.
>   **Mechanism:** Cookie value mutated → signature no longer matches.
>   **Likelihood:** Low — R2 SDKs are server-side; presigned URLs are
>   client-side fetches with no cookie interaction.
>   **Cheapest falsifier:** Capture the cookie in DevTools Application
>   tab before and after an upload; diff.
>
> - **H3:** The upload request is taking longer than the JWT's max age,
>   and the refresh path is broken.
>   **Mechanism:** Token expires mid-request; auth middleware sees an
>   expired token, treats it as a signature failure (depends on
>   implementation).
>   **Likelihood:** Medium — only if your JWT max-age is short (<10 min)
>   AND uploads are slow.
>   **Cheapest falsifier:** Check `NEXTAUTH_*` env for `JWT_MAX_AGE`;
>   correlate logout timestamps with upload durations in logs.
>
> ### 3. Recommended next experiment
> Run the H1 falsifier. If `NEXTAUTH_SECRET` differs across replicas, H1
> is confirmed and the fix is to sync the env. If it is consistent, move
> to H3.
>
> ### 4. What I would need to see
> - The output of the replica env comparison above.
> - 3-5 representative logout events from your logs with: timestamp,
>   instance/pod ID, user-agent, preceding request path.
> - Your `NEXTAUTH_*` env var list (names, not values).

## Personalisation tips

- **Production-only bugs:** if you cannot reproduce locally, add "Section
  5: How to reproduce in a controlled environment" so the model proposes
  ways to make the bug appear (load test, specific user state, time
  manipulation).
- **Heisenbug (disappears when watched):** add the line "Pay special
  attention to timing, ordering, and observer-effect hypotheses
  (race conditions, lazy initialisation, JIT optimisations)."
- **You already have one hypothesis:** state it as "Current hypothesis:
  …" before the bug report. The model will use it as H1 and generate
  alternatives, which is a cheap way to test your intuition.
- **Stop bias toward `try/catch`:** the prompt already says "do not
  propose a fix yet", but if the model still slips, append "A `try/catch`
  is not an acceptable fix; if your survivor hypothesis points to one,
  re-examine it."
