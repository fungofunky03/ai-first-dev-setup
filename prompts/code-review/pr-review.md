---
title: "PR Review (full)"
category: code-review
when_to_use: "Before approving any non-trivial PR (>200 LOC or touching shared infrastructure)"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

A structured PR review pass that surfaces correctness, design, and risk
concerns in that order. The prompt enforces a fixed output format so two
reviews of the same diff are comparable, and so reviewers do not skip the
"design" layer in favour of easier nits.

It works because it gives the model a checklist *and* asks for a verdict
with a confidence level, which discourages the "looks good to me" shaped
output models default to.

## Prompt

```
You are reviewing a pull request as a senior engineer. Be specific and
direct. Cite line numbers. Do not summarise the diff back to me.

Output exactly these five sections, in this order:

### 1. Verdict
Approve / Request changes / Block. Confidence: low / medium / high. One
sentence on why.

### 2. Correctness
Bugs, off-by-ones, null/undefined hazards, race conditions, broken error
paths. Each finding: `path:line` + what is wrong + minimal fix.

### 3. Design
Architectural concerns: leaky abstractions, premature generalisation,
violated invariants, hidden coupling. Be willing to disagree with the
author's framing.

### 4. Risk
What could go wrong in production that the tests do not cover? Migration
ordering, backward compatibility, perf at scale, observability gaps.

### 5. Nits (optional)
Naming, formatting, style. Only include if the rest is clean. Hard cap: 5
items.

Constraints:
- If a section has nothing, write "None" and move on. Do not invent issues.
- Do not suggest changes already covered by linters / formatters.
- Quote the offending line verbatim before suggesting the fix.
- If you are uncertain, say so explicitly. Do not guess.

Diff:
<paste unified diff here>
```

## Example

**Input (TypeScript / React):**

```diff
--- a/src/components/UserAvatar.tsx
+++ b/src/components/UserAvatar.tsx
@@ -1,15 +1,18 @@
 import { useEffect, useState } from "react";

 export function UserAvatar({ userId }: { userId: string }) {
-  const [src, setSrc] = useState<string>("/placeholder.png");
+  const [src, setSrc] = useState<string>();

   useEffect(() => {
-    fetch(`/api/users/${userId}/avatar`)
-      .then((r) => r.json())
-      .then((data) => setSrc(data.url));
+    const load = async () => {
+      const r = await fetch(`/api/users/${userId}/avatar`);
+      const data = await r.json();
+      setSrc(data.url);
+    };
+    load();
   }, [userId]);

-  return <img src={src} alt="avatar" />;
+  return <img src={src!} alt="avatar" />;
 }
```

**Expected output (excerpt):**

> ### 1. Verdict
> Request changes. Confidence: high. The non-null assertion masks an
> initial render where `src` is `undefined`.
>
> ### 2. Correctness
> - `src/components/UserAvatar.tsx:14` — `src!` is a lie. On the first
>   render before `useEffect` resolves, `src` is `undefined` and `<img>`
>   receives `src={undefined}`, which browsers handle inconsistently. Fix:
>   restore the placeholder default or render `null` until loaded.
> - `src/components/UserAvatar.tsx:8-11` — unchecked `fetch`. A failing
>   network call rejects the promise and is silently swallowed; the avatar
>   stays blank with no error state. Wrap in `try/catch` and surface to the
>   caller or an error boundary.
>
> ### 3. Design
> The component owns both data fetching and rendering. Consider extracting
> `useAvatarUrl(userId)` so the component is testable and reusable in
> contexts that already have the URL.
>
> ### 4. Risk
> No cancellation on unmount. If `userId` changes rapidly (list virtualisation),
> stale responses can overwrite fresh ones. Use an `AbortController`.
>
> ### 5. Nits
> None.

## Personalisation tips

- **Different audience seniority:** swap "senior engineer" for "staff
  engineer" or "tech lead" to shift the emphasis toward architecture and
  away from line-level findings.
- **Different stack:** the Design and Risk categories transfer to any
  language. Update the Correctness examples to your stack's common hazards
  (e.g. Go: nil-pointer derefs, goroutine leaks; Python: mutable defaults,
  bare `except`).
- **Team-specific concerns:** add a 6th section like "Compliance" or
  "Accessibility" if your codebase has a hard requirement that reviewers
  often miss.
- **PR-size guard:** prepend "If the diff is larger than 800 lines, refuse
  the review and ask the author to split it." This trains the team to ship
  smaller PRs.
