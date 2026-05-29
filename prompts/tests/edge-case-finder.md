---
title: "Find missing edge cases in a test file"
category: tests
when_to_use: "When you have a test file with reasonable happy-path coverage and want to find what is missing — without rewriting what is there"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Audits an existing test file against the code it tests and reports which
edge cases are absent. The non-obvious move is to separate cases that are
*missing* from cases that are *covered but should not be* (over-testing
of internals, redundant tests, framework smoke tests). Most teams have
both problems; most prompts only address the first.

## Prompt

```
Audit the test coverage of <SOURCE_FILE> against its tests in
<TEST_FILE>. Goal: identify gaps and redundancies. Do not propose code
yet.

Phase 1 — Inventory:

  a. **What the source actually does:** list every observable behaviour
     of `<SOURCE_FILE>` (public functions, error paths, side effects,
     state transitions). Cite `path:line`.
  b. **What the tests actually cover:** for each test in `<TEST_FILE>`,
     map it to which behaviour from (a) it exercises. A test can cover
     multiple behaviours.

Phase 2 — Gap analysis:

  c. **Uncovered behaviours:** items in (a) with no matching test in (b).
     For each: severity (High = error path or invariant; Medium = branch;
     Low = cosmetic).
  d. **Untested edge cases:** for each function with tests, list inputs
     that are not currently exercised. Use the same input-class taxonomy
     as `unit-tests-from-fn.md` (empty/boundary/unicode/null/negative/
     etc., scaled to the parameter types).
  e. **Untested interactions:** combinations of parameters or sequenced
     calls (`f(x); f(y)`) where order or relation matters.
  f. **Untested error paths:** every `throw`/error-return in the source
     with no test asserting it.

Phase 3 — Redundancy analysis:

  g. **Duplicate coverage:** sets of tests that exercise the same
     behaviour with no meaningful difference. Recommend keeping the
     clearest, dropping the rest.
  h. **Tests of internals:** tests that exercise private helpers, framework
     primitives, or implementation details that should be free to change.
  i. **Tests asserting on nothing:** tests that "run" the function but
     have weak or absent assertions.

Output structure:
  - Top: a 3-line summary (count of coverage gaps by severity, count of
    redundancies).
  - Then each category from (c)-(i) as a numbered list. If a category is
    empty, write "None" and move on.

Constraints:
- Cite `path:line` for every claim.
- Do not propose specific test code. The user will request that in a
  follow-up.
- Rank gaps by severity, not by ease of fixing.

Files:
<paste source and test file>
```

## Example

**Input:** `src/lib/parse-duration.ts` (from the unit-tests example) and an
existing `parse-duration.test.ts` with only 4 happy-path tests covering
`ms`, `s`, `m`, `h`.

**Expected output (excerpt):**

> Summary: 8 gaps (1 High, 6 Medium, 1 Low). 0 redundancies. 1 test
> asserts on the wrong thing.
>
> ### Uncovered behaviours
> - **High** — `parse-duration.ts:3` throws `invalid duration: ${input}`
>   on no-match. No test asserts that this throws or that the message
>   includes the input.
>
> ### Untested edge cases
> - **Medium** — empty string input (`""`). No test.
> - **Medium** — unsupported unit (`"5d"`). No test.
> - **Medium** — number without unit (`"30"`). No test.
> - **Medium** — unit without number (`"s"`). No test.
> - **Medium** — leading/trailing whitespace. The regex is anchored,
>   so these should throw, but there is no test guaranteeing that.
> - **Medium** — case sensitivity (`"30S"` vs `"30s"`). Behaviour is
>   "throws" per the regex, but undocumented and untested.
> - **Low** — zero (`"0s"`). The function accepts it; if that is
>   intentional, add a test.
>
> ### Untested interactions
> None — single-parameter function.
>
> ### Untested error paths
> See "Uncovered behaviours" — the single throw at line 3.
>
> ### Duplicate coverage
> None.
>
> ### Tests of internals
> None.
>
> ### Tests asserting on nothing
> - `parse-duration.test.ts:14` — `it("parses milliseconds")` calls
>   `parseDuration("500ms")` but does not assert the return value.
>   (Was probably meant to be `expect(parseDuration("500ms")).toBe(500)`.)

## Personalisation tips

- **Mutation testing alignment:** if you use Stryker / mutmut, append
  "Prioritise gaps that would be detected by a mutation testing run
  (i.e. surviving mutants on this file)."
- **Strict TypeScript stacks:** for projects with `strict: true`, many
  "what if X is null" cases are compile-time impossible. Add "Assume
  TypeScript strict mode; do not propose tests for cases the type system
  rules out at compile time."
- **Integration vs unit boundary:** if the source file is integration-ish
  (does I/O), add "Cases that would require mocking shared infrastructure
  (DB, network) go under 'Integration test recommendations' instead, not
  in the unit-test gaps."
- **Quick triage mode:** for huge test files where you only want the top
  3 problems, append "Output only the 3 highest-severity items. Skip the
  rest." Useful in a PR review context.
