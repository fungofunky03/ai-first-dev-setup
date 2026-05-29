---
title: "Migrate pattern across codebase"
category: refactor
when_to_use: "When you need to change every instance of pattern X to pattern Y across many files (e.g. class components → hooks, callbacks → async/await, deprecated API → new API)"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Codebase-wide migrations are where AI assistants either save the most time
or cause the most damage. This prompt forces an explicit migration plan
with a single canonical example *before* any files are touched, then
applies the pattern mechanically and lists every change for review.

The non-obvious move is requiring the model to identify the *edge cases*
that look like the source pattern but should *not* be migrated. Skipping
this step is the most common cause of subtle regressions.

## Prompt

```
You are performing a codebase-wide migration from <OLD_PATTERN> to
<NEW_PATTERN>.

Phase 1 — Plan (output only, no edits yet):

  a. **Canonical example:** Show one file converted from old to new, with
     a short note on what changed and why.
  b. **Detection signature:** a short description (or regex if practical)
     for finding instances of <OLD_PATTERN>.
  c. **Edge cases:** code that *looks* like <OLD_PATTERN> but must be
     left alone. For each: an example and the rule that excludes it.
  d. **Migration steps:** the exact sequence you will apply per file.
  e. **Verification:** the command(s) the user runs after each batch to
     confirm no regression (typecheck, test, lint).

Stop here. Wait for the user to approve the plan or amend it.

Phase 2 — Apply (only after explicit approval):

  - Migrate in batches of at most 5 files. After each batch, list:
    - Files touched (with `path:line` for non-trivial changes).
    - Files skipped, with the reason.
    - Anything that triggered an edge-case rule.
  - Pause between batches for the user to run verification.

Constraints:
- Do not delete the old API/helper/utility unless explicitly told to. Keep
  it available during the migration and flag it for removal at the end.
- If a file has both clean instances of <OLD_PATTERN> and edge cases, ask
  before touching it.
- Never auto-format or restyle code that is not part of the migration.

Files to migrate (grep this list yourself if not provided):
<list of paths, or "find them in src/" with a starting hint>
```

## Example

**Input:** `OLD_PATTERN` = "React class components with state", `NEW_PATTERN`
= "function components with hooks", scope = "src/components/**/*.tsx".

**Phase 1 output (excerpt):**

> **Canonical example:**
>
> ```diff
> -class Counter extends React.Component<{}, { count: number }> {
> -  state = { count: 0 };
> -  increment = () => this.setState({ count: this.state.count + 1 });
> -  render() {
> -    return <button onClick={this.increment}>{this.state.count}</button>;
> -  }
> -}
> +function Counter() {
> +  const [count, setCount] = useState(0);
> +  return <button onClick={() => setCount(count + 1)}>{count}</button>;
> +}
> ```
>
> **Detection signature:** files containing `extends React.Component` or
> `extends Component`, with class declarations that have a `render()`
> method.
>
> **Edge cases (do NOT migrate):**
> - Class components that use `componentDidCatch` (error boundaries) —
>   error boundaries still require classes in React 18.
> - Class components inside `src/legacy/` — vendor-frozen code per
>   `README.md`.
> - Classes that extend `PureComponent` and have a custom
>   `shouldComponentUpdate` returning anything other than `true` — needs
>   manual review for `React.memo` equivalence.
>
> **Migration steps per file:**
> 1. Convert state to `useState` per field.
> 2. Convert `componentDidMount` / `componentWillUnmount` to `useEffect`.
> 3. Convert instance methods to inline arrow functions or `useCallback`
>    if passed to memoised children.
> 4. Convert refs (`this.myRef`) to `useRef`.
> 5. Update parent imports if a default export was the class itself.
>
> **Verification (after each batch):**
> - `npm run typecheck`
> - `npm test -- --testPathPattern=src/components`
>
> Files in scope: 47. Files with edge cases (paused for your decision):
> `src/components/ErrorBoundary.tsx`, `src/legacy/...` (3 files).
>
> Approve to start Batch 1?

## Personalisation tips

- **API deprecation:** swap `OLD_PATTERN`/`NEW_PATTERN` for `oldApi(args)` /
  `newApi(args)`. The structure is identical; the canonical example becomes
  the documentation for the new API.
- **Multi-language migration:** if the migration spans languages
  (e.g. removing a deprecated SDK across TS + Python), run two separate
  passes with the prompt — do not try to combine them.
- **Read-only first pass:** run Phase 1 alone, save the output to your PR
  description, and discuss the edge cases with the team before any edits.
  This is the cheap version of an RFC.
- **Compilation gates:** if your CI runs typecheck per-batch in seconds,
  drop the "pause between batches" requirement to speed things up. Keep it
  if verification is slow.
