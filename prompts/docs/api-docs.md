---
title: "Generate API documentation"
category: docs
when_to_use: "When you need reference docs for a library's public API, or REST/RPC endpoints — from the source of truth (code), not a hand-written copy"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Generates API reference docs directly from code. Two flavours: library APIs
(public exports of a package) and HTTP APIs (routes/handlers). Both follow
the same principle — the documentation is a *projection* of the code, so
you generate it from the code and regenerate when the code changes.

The prompt enforces a strict per-endpoint or per-export template so output
is parseable: easy to re-run, easy to diff against the previous version,
and easy to lint for completeness.

## Prompt

```
Generate API reference documentation for <SCOPE>.

<SCOPE> is one of:
  (a) "the public exports of <PACKAGE>" — for library APIs.
  (b) "the HTTP routes in <PATH_GLOB>" — for REST/RPC APIs.

Phase 1 — Inventory:
  - For (a): list every export from the package's entry point(s). Group
    by category (functions, classes, types, constants).
  - For (b): list every route discovered. Columns: method, path, handler
    file:line, auth requirement (read from middleware / decorators).

Phase 2 — Per-item documentation. For each, use exactly this template:

  ### <Name>
  <One-sentence summary in present tense. "Returns…", "Creates…", not
  "This function returns…".>

  **Signature** (for library) or **Endpoint** (for HTTP):
  ```<language>
  <copy verbatim from code, including types / decorators>
  ```

  **Parameters / Request:**
  Table. Columns: name, type, required, description, constraints.

  **Returns / Response:**
  Type/shape. For HTTP, status codes and example body per status.

  **Errors:**
  Every distinct error this can produce. For HTTP, status code + when.
  For library, exception type + when.

  **Example:**
  Minimal working example. For library, a 3-10 line code snippet. For
  HTTP, a `curl` command + sample response.

  **Notes:**
  Anything else: rate limits, deprecation, side effects, idempotency.
  Skip if none.

Constraints:
- Read the code. Do not infer parameter descriptions from names — read
  the JSDoc/docstring/comment, or write "(undocumented in source)" and
  list it under coverage gaps at the end.
- Do not invent endpoints, parameters, or error codes.
- If a parameter has a default, state it explicitly.
- For HTTP APIs, include the auth requirement on every endpoint, even
  the public ones (write "Public" explicitly).

After generation:
- "Coverage gaps" — exports/routes you saw but could not fully document
  (and why).
- "Suggested doc improvements in source" — files that would benefit from
  docstrings on undocumented public items.
```

## Example

**Input:** "the HTTP routes in `src/api/**/*.ts`" for a small Hono server.

**Expected output (excerpt):**

> ### POST /api/projects
> Creates a new project owned by the authenticated user.
>
> **Endpoint:**
> ```ts
> app.post("/api/projects", requireAuth, async (c) => { ... })
> // src/api/projects.ts:14
> ```
>
> **Request:** `application/json`
>
> | Field         | Type     | Required | Description                       | Constraints              |
> | ------------- | -------- | -------- | --------------------------------- | ------------------------ |
> | `name`        | string   | yes      | Project display name              | 1-80 chars               |
> | `description` | string   | no       | Free-form description             | ≤ 1000 chars             |
> | `visibility`  | "public" \| "private" | no | Default: `"private"` | enum               |
>
> **Response:**
> - `201 Created`
>   ```json
>   { "id": "proj_abc123", "name": "My project", "createdAt": "…" }
>   ```
> - `400 Bad Request` — validation failure
> - `401 Unauthorized` — missing or invalid session
> - `409 Conflict` — a project with this name already exists for this user
>
> **Errors:**
> - 400 from Zod schema in `src/api/projects.ts:18`
> - 409 from unique constraint on `(owner_id, name)` in
>   `migrations/0007_projects.sql`
>
> **Example:**
> ```bash
> curl -X POST https://api.example.com/api/projects \
>   -H "Authorization: Bearer $TOKEN" \
>   -H "Content-Type: application/json" \
>   -d '{"name":"My project"}'
> ```
>
> **Notes:** Idempotent within a 60-second window keyed by
> `(user, name)` — `src/api/projects.ts:22-28`.
>
> [...rest of routes...]
>
> Coverage gaps:
> - `POST /api/projects/:id/transfer` — handler delegates to a service
>   in a file not provided; cannot document error cases.
>
> Suggested doc improvements in source:
> - `src/api/projects.ts:14` — handler has no comment explaining the
>   idempotency window.

## Personalisation tips

- **Spec-first stacks:** if your project uses OpenAPI / gRPC / GraphQL
  schemas, prefer documenting from the spec, not the handlers. Append
  "Read the OpenAPI document at <PATH> and document each operation."
- **Output format:** swap the markdown template for OpenAPI YAML, Stripe-style
  reference, or Slate-style if your docs site uses a fixed format.
- **Stale-detection mode:** run this against `main` and `HEAD~50`, diff the
  outputs to find APIs that changed without a corresponding docs update.
- **Multi-tenant authn:** if your auth model has roles/scopes, add
  "Document the minimum scope required for each endpoint."
