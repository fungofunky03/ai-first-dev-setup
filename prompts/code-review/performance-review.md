---
title: "Performance review"
category: code-review
when_to_use: "Before merging changes to hot paths, data access layers, or anything called per-request or per-render"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

A performance-focused pass that distinguishes between *measured* problems
and *suspected* ones. Most performance reviews fail by optimising things
that were not slow; this prompt forces the model to estimate big-O,
allocation, and I/O cost first, then rank findings by likely impact rather
than ease of optimisation.

## Prompt

```
You are reviewing a diff for performance defects. Focus on the dimensions
that actually matter: time complexity, allocations, I/O, and contention.
Ignore micro-optimisations unless the surrounding code path is genuinely
hot.

For each function/block in the diff, classify:
  - **Time:** O(?) in terms of the input size that matters.
  - **Allocations:** per call. Constant, linear, or worse?
  - **I/O:** none, single, N+1, network round-trips per call.
  - **Contention:** locks held, shared state mutated, transactions opened.

Then output two sections:

### Performance findings
Ordered by likely impact (assume the code is on a request path unless told
otherwise). Each finding:
  - **Impact:** High / Medium / Low — and *why* you think so.
  - **Location:** `path:line`
  - **Problem:** what is slow and the dimension (time / alloc / I/O /
    contention).
  - **Fix:** the minimal change. Estimate the expected speedup if you can.
  - **Measurement:** what command/tool would confirm the improvement
    (`hyperfine`, `pprof`, browser perf tab, `EXPLAIN ANALYZE`).

### Non-problems
Things that *look* like performance issues but are not on this path's
critical budget. State why, briefly. This section prevents wasted work.

Constraints:
- Do not flag list comprehensions, map/filter, or other idiomatic code as
  "slow" unless input sizes warrant it.
- N+1 query patterns are always High impact in a request path.
- If you cannot infer input size, say so and ask.

Diff and context:
<paste here>
```

## Example

**Input:**

```ts
// src/api/dashboard.ts
import { db } from "../db";

export async function getDashboard(userId: string) {
  const user = await db("users").where({ id: userId }).first();
  const projects = await db("projects").where({ owner_id: userId });

  const projectsWithStats = await Promise.all(
    projects.map(async (p) => {
      const taskCount = await db("tasks").where({ project_id: p.id }).count();
      const lastActivity = await db("events")
        .where({ project_id: p.id })
        .orderBy("created_at", "desc")
        .first();
      return { ...p, taskCount, lastActivity };
    })
  );

  return { user, projects: projectsWithStats };
}
```

**Expected output (excerpt):**

> Classification:
> - `getDashboard`: Time O(N) in projects per user; Allocations O(N);
>   I/O 1 + 1 + 2·N round-trips; Contention none.
>
> ### Performance findings
>
> - **Impact:** High — request-path function, 2 extra queries per project,
>   typical user has 10-50 projects → 20-100 extra round-trips. On a
>   100ms-RTT DB connection this is 2-10s of avoidable latency.
> - **Location:** `src/api/dashboard.ts:9-15`
> - **Problem:** Classic N+1. Each project triggers two additional queries
>   (`taskCount`, `lastActivity`) inside `Promise.all`.
> - **Fix:** Two batch queries.
>   - `db("tasks").whereIn("project_id", ids).groupBy("project_id").count()`
>   - `db("events").whereIn("project_id", ids).orderBy(...).distinctOn("project_id")`
>     (or window function if your DB supports it).
>   Then zip results back into projects in memory. Expected speedup: ~10-50x.
> - **Measurement:** `EXPLAIN ANALYZE` on the rewritten queries; compare
>   total time in your request logs.
>
> ### Non-problems
> - The `Promise.all` itself is fine; the issue is the number of queries,
>   not the concurrency primitive.
> - `db("users").first()` is a single point lookup on a primary key. Not a
>   bottleneck.

## Personalisation tips

- **Frontend perf:** swap "request path" for "render path" and the
  dimensions become re-renders, layout thrashing, and bundle bytes. Add
  "What re-renders this triggers" to the classification.
- **Batch jobs:** add "throughput" to the classification dimensions. Latency
  is less important than items/second for nightly jobs.
- **DB-heavy systems:** ask the model to annotate each query with a guess
  at which index would be used. Often surfaces missing indexes.
- **Already-instrumented code:** include sample timings from your APM (e.g.
  "this function takes 800ms p95") so the model can prioritise correctly.
