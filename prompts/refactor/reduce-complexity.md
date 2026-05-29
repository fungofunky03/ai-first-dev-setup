---
title: "Reduce cyclomatic complexity"
category: refactor
when_to_use: "When a function has nested conditionals, multiple early returns mixed with else branches, or a complexity metric flagged it"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Walks a function from its current control-flow shape to a flatter, more
linear one using guard clauses, early returns, and small helper extractions.
The prompt is opinionated about *how* — favouring guard clauses over nested
ifs and avoiding the "extract method" trap where a function is split into
helpers that can only be understood together.

## Prompt

```
Reduce the cyclomatic complexity of <FUNCTION_NAME> in <FILE>. Preserve
behaviour exactly.

Techniques, in priority order:
1. Guard clauses: replace `if (good) { ... } else { return errorCase; }`
   with `if (!good) return errorCase; ...`.
2. Combine adjacent conditions that lead to the same outcome.
3. Replace nested ternaries with named intermediate variables or early
   returns.
4. Extract a helper ONLY when (a) the extracted block has a clear name,
   (b) it has at most 3 inputs and 1 output, and (c) it can be understood
   without reading the caller. Otherwise, leave the code inline.
5. Replace state-machine-shaped conditionals with a lookup table or a
   small enum if there are more than 3 branches.

Do not:
- Change the function signature.
- Rename variables unless their current names actively mislead.
- Introduce new abstractions (classes, interfaces) for a single function.
- Add comments explaining what the code does. Add a comment only for
  WHY a non-obvious choice was made (e.g. a workaround).

Output:
- The refactored function in full.
- A bullet list of the transformations applied, in the order applied.
- The before/after cyclomatic complexity (count of independent paths).

Function:
<paste function and any helpers it calls>
```

## Example

**Input:**

```ts
function priceShipping(order: Order, user: User): number {
  if (order.items.length > 0) {
    if (user.isPremium) {
      if (order.total >= 50) {
        return 0;
      } else {
        if (order.weight < 5) {
          return 3;
        } else {
          return 6;
        }
      }
    } else {
      if (order.total >= 100) {
        return 0;
      } else {
        if (order.weight < 5) {
          return 5;
        } else {
          return 10;
        }
      }
    }
  } else {
    throw new Error("empty order");
  }
}
```

**Expected output:**

```ts
function priceShipping(order: Order, user: User): number {
  if (order.items.length === 0) throw new Error("empty order");

  const freeThreshold = user.isPremium ? 50 : 100;
  if (order.total >= freeThreshold) return 0;

  const isLight = order.weight < 5;
  if (user.isPremium) return isLight ? 3 : 6;
  return isLight ? 5 : 10;
}
```

> Transformations:
> 1. Guard clause for empty order — eliminates the outer `if/else`.
> 2. Lifted `freeThreshold` so the free-shipping check is a single line
>    regardless of premium status.
> 3. Lifted `isLight` so the four leaf branches collapse into two ternaries.
>
> Cyclomatic complexity: before 6, after 4.

## Personalisation tips

- **Strict complexity target:** append "Target cyclomatic complexity of
  <N>. If you cannot reach it without sacrificing readability, stop and
  explain." Useful when integrating with a linter that enforces a hard cap.
- **State machine bias:** for functions that look like state machines
  (auth flows, payment states), add "If the branching is state-shaped,
  consider modelling it as an explicit state enum with a transition map."
- **Skip extraction:** if you find the model extracts helpers too
  aggressively, add "Do not extract any helper functions. Reduce complexity
  in place." Sometimes flatter is genuinely better.
- **Other languages:** the rules transfer 1:1 to Python and Go. Replace
  ternaries with conditional expressions / `if/else` blocks accordingly.
