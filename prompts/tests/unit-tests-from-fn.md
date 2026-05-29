---
title: "Unit tests from a function"
category: tests
when_to_use: "When you need a first-pass test suite for a single function, especially before refactoring it"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Generates a unit-test suite from one function, enumerating cases by input
class rather than by code path. Path-coverage-driven tests miss bugs the
function has but the code does not express yet; input-class tests catch
both. The prompt also asks the model to declare *what it is not testing*
(integration, performance, etc.) so the suite has a clear boundary.

## Prompt

```
Generate a unit-test suite for <FUNCTION_NAME> in <FILE>. Use
<TEST_FRAMEWORK>.

Phase 1 — Test case inventory (output before any code):

  a. Read the function. Identify each input parameter and its type.
  b. For each parameter, enumerate **input classes**: groups of values
     the function should treat equivalently or distinctly. Examples:
     for a string param — empty, single char, max length, contains
     unicode, contains whitespace; for a number — zero, positive,
     negative, max safe integer, NaN, Infinity.
  c. Identify **interactions** between parameters: combinations where the
     behaviour is non-trivial (e.g. `start > end`, both null, one null).
  d. Identify **error cases**: inputs that should throw or return an
     error.
  e. Identify **observable side effects** (if any): DB writes, network
     calls, mutations of inputs.

Output as a numbered list. Each item: one-line description + expected
behaviour.

Stop here. Wait for the user to:
  - Approve the list,
  - Add cases you missed,
  - Or remove cases that are out of scope.

Phase 2 — Test code (only after approval):

  - One `it(...)` / `test(...)` per case from the approved list.
  - Test names follow "should <behaviour> when <input class>" or the
    framework's convention if different.
  - Arrange / Act / Assert structure with blank lines between sections.
  - No setup helpers for a 1-case suite. Add a `beforeEach` only when
    ≥3 tests share setup.
  - For side effects, use the project's existing test doubles strategy
    (spies, fakes, in-memory implementations). Do not introduce a new
    mocking library.

Constraints:
- Do not test private helpers that the function uses. Test only the
  public function.
- Do not test the framework / language (e.g. don't test that `Array.map`
  works).
- Output the list of cases you deliberately did NOT test (e.g. concurrent
  access, perf) under "Out of scope".

Function:
<paste function + types it depends on>
```

## Example

**Input function:**

```ts
// src/lib/parse-duration.ts
export function parseDuration(input: string): number {
  const match = input.match(/^(\d+)(ms|s|m|h)$/);
  if (!match) throw new Error(`invalid duration: ${input}`);
  const value = parseInt(match[1], 10);
  const unit = match[2] as "ms" | "s" | "m" | "h";
  const multipliers = { ms: 1, s: 1000, m: 60_000, h: 3_600_000 };
  return value * multipliers[unit];
}
```

**Phase 1 output:**

> Input parameter: `input: string`. Input classes:
>
> 1. Valid `ms`: `"500ms"` → 500.
> 2. Valid `s`: `"30s"` → 30_000.
> 3. Valid `m`: `"5m"` → 300_000.
> 4. Valid `h`: `"2h"` → 7_200_000.
> 5. Zero: `"0s"` → 0. (Edge: is zero allowed? Function permits it.)
> 6. Large number: `"99999h"` → no overflow within Number.MAX_SAFE_INTEGER.
> 7. Empty string → throws.
> 8. Number without unit: `"30"` → throws.
> 9. Unit without number: `"s"` → throws.
> 10. Unsupported unit: `"5d"` → throws.
> 11. Leading whitespace: `" 30s"` → throws (regex is anchored).
> 12. Trailing whitespace: `"30s "` → throws.
> 13. Negative: `"-5s"` → throws (regex requires `\d+`).
> 14. Decimal: `"1.5s"` → throws.
> 15. Mixed case: `"30S"` → throws (regex is case-sensitive).
>
> Interactions: none (single parameter).
> Side effects: none.
>
> Out of scope:
> - Performance under 10^6 calls/sec.
> - Behaviour on non-string inputs — TypeScript prevents it at the call
>   site; this is a runtime function, but enforcing at compile time is
>   the chosen boundary.

**Phase 2 output (excerpt, Vitest):**

```ts
// src/lib/parse-duration.test.ts
import { describe, expect, it } from "vitest";
import { parseDuration } from "./parse-duration";

describe("parseDuration", () => {
  it("parses milliseconds", () => {
    expect(parseDuration("500ms")).toBe(500);
  });

  it("parses seconds", () => {
    expect(parseDuration("30s")).toBe(30_000);
  });

  it("parses minutes", () => {
    expect(parseDuration("5m")).toBe(300_000);
  });

  it("parses hours", () => {
    expect(parseDuration("2h")).toBe(7_200_000);
  });

  it("allows zero", () => {
    expect(parseDuration("0s")).toBe(0);
  });

  it("throws on empty string", () => {
    expect(() => parseDuration("")).toThrow("invalid duration");
  });

  it("throws when unit is missing", () => {
    expect(() => parseDuration("30")).toThrow("invalid duration");
  });

  // ...remaining cases...
});
```

## Personalisation tips

- **Property-based testing:** if the project uses fast-check / Hypothesis,
  append "After the example-based tests, add 2-3 property-based tests
  for general invariants (e.g. round-trip with a `formatDuration`)."
- **Coverage-driven:** if you must hit a coverage number, add "After
  Phase 2, run the project's coverage tool and add tests to reach
  <PERCENT>% on this file specifically." Mention the tool by name.
- **Behaviour-driven naming:** swap "should X when Y" for the project's
  existing convention if different (e.g. Gherkin-style, "given/when/then").
- **Existing tests:** if a test file already exists, prepend "Do not
  duplicate cases already covered in <EXISTING_TEST_FILE>. Read it first,
  list what it covers, and only add the gap."
- **Different language:** the workflow transfers 1:1. For Python use
  `pytest` style; for Go, table-driven tests with sub-tests.
