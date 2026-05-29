---
title: "Generate E2E test scenarios from a user flow"
category: tests
when_to_use: "When you need end-to-end test coverage for a critical user journey (sign-up, checkout, onboarding) and want to enumerate scenarios before writing code"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Generates an E2E test plan as a *scenario list* before any test code is
written. The mistake teams make with E2E tests is going straight to
Playwright/Cypress code, ending up with a thicket of brittle, slow tests
that all exercise the same happy path with cosmetic differences. This
prompt forces enumeration of the journey's decision points first.

It produces Playwright code by default but the structure adapts trivially
to Cypress, Selenium, or even manual test scripts.

## Prompt

```
Plan E2E test coverage for the user journey: <JOURNEY>.

Phase 1 — Scenario enumeration:

Map the journey as a small graph: each node is a user action or system
response, each edge is the user choice or outcome that leads to the
next. Output:

  a. **Happy path:** the most common successful traversal, step by step.
  b. **Branch points:** nodes where the user can choose, or the system
     can behave differently. For each, list every observable outcome.
  c. **Failure paths:** scenarios where the user encounters an error or
     dead end. For each: trigger, expected feedback, expected recovery.
  d. **Cross-cutting concerns:** browser back/forward, refresh, network
     loss, multiple tabs, copy-pasting URLs into a new session. For each,
     state whether it is in scope for this suite.

Output as a numbered scenario list. Each scenario: name, preconditions,
steps, assertions. Estimate runtime in seconds (rough order of magnitude).

Stop. Wait for the user to:
  - Trim the list to what matters,
  - Add scenarios you missed.

Phase 2 — Test code in <FRAMEWORK> (default: Playwright + TypeScript):

  - One test file per scenario group (e.g. all "checkout" scenarios in
    one file).
  - Use Page Object Model only if there are ≥3 tests touching the same
    page. Otherwise inline.
  - Selectors: prefer `getByRole`, `getByLabel`, `getByTestId`. No XPath
    or CSS selectors except as last resort. If `data-testid` does not
    exist where needed, flag for the developer.
  - Each test is independent: it can run alone, in any order, in parallel.
  - No `sleep()`. Use the framework's wait-for-condition primitives.

Constraints:
- E2E tests are expensive. Cap at <MAX_SCENARIOS> scenarios in the first
  pass.
- Do not test backend behaviour that has unit/integration coverage. Test
  only what unit tests cannot: real browser, real network, real user
  interaction.
- For flaky-prone areas (animations, third-party iframes), state up front
  that they will need explicit handling.

Journey:
<describe the journey: starting state, user goal, ending state>
```

## Example

**Input:**

> `JOURNEY`: A signed-out user signs up with email/password, verifies
> email, completes onboarding (3 questions), lands on the dashboard.
> `FRAMEWORK`: Playwright + TypeScript.
> `MAX_SCENARIOS`: 8.

**Phase 1 output (excerpt):**

> **Happy path:**
> 1. Visit `/signup`.
> 2. Enter email + password (≥10 chars).
> 3. Submit. See "Check your email" confirmation.
> 4. Click verification link (intercept email or use test inbox).
> 5. Land on `/onboarding/step-1`. Answer Q1. Continue.
> 6. Answer Q2. Continue.
> 7. Answer Q3. Submit.
> 8. Land on `/dashboard` with welcome banner.
>
> **Branch points:**
> - Step 2 password validation: too short, no number, no symbol — each
>   shows the corresponding error inline.
> - Step 5/6/7: each onboarding question may be skipped via a "Skip"
>   button → skipped questions default to `null` in the user profile.
>
> **Failure paths:**
> - Email already registered → "An account with this email exists. Sign
>   in?" with a link to `/signin`.
> - Verification link expired → page shows "Resend verification" CTA.
> - Onboarding submit fails (network) → toast "Could not save, retry"
>   and the user stays on the page with answers preserved.
>
> **Cross-cutting:**
> - Refresh mid-onboarding: in scope. Answers should persist (server-side).
> - Open verification link in a different browser: in scope. Should still
>   land on `/onboarding/step-1` correctly.
> - Back button during onboarding: out of scope for v1 (not implemented
>   to spec yet).
>
> Scenarios (capped at 8):
> 1. **Happy path signup-to-dashboard** (preconditions: clean test
>    inbox; ~25s).
> 2. **Sign-up with already-registered email** (~8s).
> 3. **Password validation errors** (parametrised: too-short, no-number,
>    no-symbol; ~10s total).
> 4. **Verification link expired** (~10s).
> 5. **Skip all onboarding questions** (~20s).
> 6. **Onboarding submit retry after network failure** (~15s, requires
>    network intercept).
> 7. **Refresh mid-onboarding preserves answers** (~18s).
> 8. **Verification in a different browser session** (~25s).

**Phase 2 output (excerpt):**

```ts
// tests/e2e/signup.spec.ts
import { test, expect } from "@playwright/test";
import { newTestInbox } from "./helpers/test-inbox";

test("happy path: sign up → verify → onboard → dashboard", async ({ page }) => {
  const inbox = await newTestInbox();
  const password = "Str0ngPass!word";

  await page.goto("/signup");
  await page.getByLabel("Email").fill(inbox.address);
  await page.getByLabel("Password").fill(password);
  await page.getByRole("button", { name: "Create account" }).click();

  await expect(page.getByText(/check your email/i)).toBeVisible();

  const verificationUrl = await inbox.waitForVerificationLink();
  await page.goto(verificationUrl);

  await expect(page).toHaveURL(/\/onboarding\/step-1/);
  // ...continue through onboarding...

  await expect(page).toHaveURL("/dashboard");
  await expect(page.getByText(/welcome/i)).toBeVisible();
});
```

## Personalisation tips

- **Cypress / Selenium / Webdriver.io:** the structure is identical; only
  the selector and waiter API changes. Set `FRAMEWORK` accordingly.
- **Mobile / React Native:** add "Use Detox/Appium semantics. Replace
  `getByRole` with the framework's accessibility-id queries."
- **Authentication-heavy apps:** add "Most scenarios should start from a
  logged-in state via a stored auth cookie / token, not the login UI.
  Only the auth scenarios go through the UI login."
- **CI cost budget:** add "Total suite must run in ≤<MINUTES> minutes
  with default parallelism. Cut scenarios that push past the budget,
  ranked by uniqueness of coverage."
- **Visual regression:** out of scope for this prompt. Pair with a
  dedicated visual-regression tool (Percy, Chromatic).
