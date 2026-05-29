---
title: "Security audit (focused)"
category: code-review
when_to_use: "Before merging changes that touch auth, input handling, cryptography, file I/O, or external API calls"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

A focused pass that looks only for security issues, not general code quality.
Separating security from general review matters: in a mixed review, the model
treats a SQL injection and a naming nit as comparable items in a list. This
prompt forces severity-first ordering and refuses to surface anything below a
real threat.

It also asks the model to state what it *did not* check, which prevents false
confidence from a clean report.

## Prompt

```
You are performing a security review. Look only for security defects. Ignore
style, performance, and general design unless they create a security risk.

Output exactly these sections:

### Findings
One block per finding, ordered by severity (Critical → High → Medium → Low).
Each block:

  - **Severity:** Critical / High / Medium / Low
  - **Class:** OWASP-style category (e.g. Injection, Broken Access Control,
    Insecure Deserialisation, SSRF, Sensitive Data Exposure)
  - **Location:** `path:line`
  - **What:** the vulnerable code, quoted verbatim
  - **Why:** how it can be exploited, concretely
  - **Fix:** the minimal change that removes the risk

### Not in scope
What you deliberately did not check (e.g. "I did not analyse the build
pipeline", "I did not check third-party dependencies for known CVEs").

### Coverage gaps
Areas of the diff where you could not reach a verdict because the relevant
context is missing (e.g. "I cannot tell if `validateUser` sanitises input
because it is imported from a file not included in the diff").

Constraints:
- Critical/High require a concrete exploit path. Speculative risk is Medium
  or Low.
- Do not flag missing defence-in-depth if the primary control is sound.
- If the diff is clean, write "No findings" under Findings. Do not invent
  issues.

Diff (and supporting files if needed):
<paste here>
```

## Example

**Input:**

```ts
// src/api/search.ts
import { db } from "../db";

export async function searchUsers(req: Request) {
  const q = new URL(req.url).searchParams.get("q") ?? "";
  const rows = await db.raw(
    `SELECT id, email FROM users WHERE name LIKE '%${q}%'`
  );
  return Response.json(rows);
}
```

**Expected output (excerpt):**

> ### Findings
>
> - **Severity:** Critical
> - **Class:** SQL Injection
> - **Location:** `src/api/search.ts:6-8`
> - **What:** `` `SELECT id, email FROM users WHERE name LIKE '%${q}%'` ``
> - **Why:** `q` is user-controlled (`?q=` query param) and interpolated
>   directly into a raw SQL string. A request like
>   `?q=%25';DROP TABLE users;--` executes attacker-controlled SQL with
>   whatever privileges the DB user has.
> - **Fix:** use parameterised queries. Replace `db.raw(...)` with
>   `db("users").where("name", "like", `%${q}%`).select("id", "email")` or
>   the equivalent parameterised raw query.
>
> ### Not in scope
> - Authn/authz: there is no auth check on this endpoint, but the diff
>   does not show whether middleware enforces it upstream.
> - Rate limiting: not visible in this file.
>
> ### Coverage gaps
> - `../db` is not in the diff; I cannot confirm whether `db.raw` already
>   escapes via some wrapper. The recommendation assumes it does not.

## Personalisation tips

- **Compliance overlay:** if you operate under SOC2 / HIPAA / PCI, append
  "Tag each finding with the relevant control IDs from `<FRAMEWORK>`."
- **Language-specific hazards:** prepend a line like "Pay particular
  attention to Python pickle/deserialisation issues" or "Pay particular
  attention to Go template injection" to bias the model toward your
  language's common pitfalls.
- **Trust model:** if some inputs are trusted (e.g. internal RPC from
  authenticated services), state the trust boundary explicitly to avoid
  false positives.
- **Pair with a SAST tool:** this prompt is not a replacement for Semgrep /
  CodeQL. Use it for issues those tools miss (logic flaws, custom auth).
