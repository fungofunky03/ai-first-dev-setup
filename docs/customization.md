# Customization

Per-stack adaptation guide for the rules, prompts, and workflows.
The defaults bias toward TypeScript / React (the most common stack
among the audience for this repo); the structural pieces work
unchanged across languages, but a few details deserve a per-stack
override.

## TypeScript / JavaScript

The defaults are written for TS. The only customisations worth doing:

- **Framework specifics in `CLAUDE.md`:** name your meta-framework
  (Next.js, Remix, SvelteKit, etc.) and its conventions explicitly.
  "Use React Server Components by default" or "Use the App Router, not
  the Pages Router" prevents a class of confusion.
- **Tooling alignment in `.cursorrules`:** if your project uses Biome,
  ESLint, or Prettier, add a line referencing the config file so the
  AI doesn't reformat code differently from your formatter.
- **Type imports:** if you use the `import type` syntax, add "Always
  use `import type` for type-only imports" to `.cursorrules`. Models
  guess wrong about half the time.
- **Monorepos:** in `CLAUDE.md`, name the workspace topology. "This is
  a pnpm workspace with packages under `packages/*` and apps under
  `apps/*`. Always specify the full package name when referencing code."

### Prompt example tweaks

The default prompt examples are TS/React. They transfer 1:1 to other
JS frameworks; just be aware that `getByRole` etc. are Playwright /
Testing Library specific.

## Python

Replace the TS-specific clauses in `.cursorrules` (TypeScript section)
with these:

- "All public functions and methods have type hints. `from __future__
  import annotations` at the top of every module on 3.9+."
- "Use `pyright` strict (or `mypy --strict`) as the type-check standard."
- "Prefer `pydantic` BaseModel over `dataclass` for anything that
  crosses an API boundary; dataclass is fine for purely internal data."
- "Match the project's package manager (`uv`, `poetry`, `pip`); do
  not introduce a new one."

In `CLAUDE.md`:

- Specify the Python version explicitly (`3.11`, `3.12`). Behavior
  differs.
- Specify the async stack if any (asyncio, anyio, trio).
- Specify your testing framework — pytest is dominant but not universal.

For prompts:

- `prompts/tests/unit-tests-from-fn.md` — works as-is; the model will
  use pytest fixtures and parametrize patterns.
- `prompts/refactor/reduce-complexity.md` — Python has match statements
  in 3.10+; add "Prefer `match` over chained `if/elif` when matching
  more than 3 distinct patterns" to the personalisation.

## Go

Replace the TS-specific clauses with:

- "Always handle errors explicitly. `if err != nil { return fmt.Errorf("doing X: %w", err) }`
  with context."
- "Define interfaces at the consumer, not the producer."
- "Use `context.Context` as the first parameter for any function doing
  I/O or with cancellation semantics."
- "Match the project's linter (`golangci-lint`); do not introduce a
  new one."
- "Use table-driven tests with `t.Run` for sub-tests."

In `CLAUDE.md`:

- Specify the Go version. Generics behaviour and stdlib additions
  shift per release.
- Name the module structure (`cmd/`, `internal/`, `pkg/`) and which is
  the public surface.

For prompts:

- `prompts/tests/unit-tests-from-fn.md` — the model defaults to TS
  examples. Add: "Use Go table-driven test style with `t.Run` and
  per-case structs. Place tests in `<file>_test.go`."
- `prompts/refactor/extract-component.md` — for Go, "component"
  becomes "package-level function or type". The rules are otherwise
  identical.

## Rust, Kotlin, Swift, other typed languages

Apply the structural rules unchanged (plan-confirm-execute,
anti-patterns, error handling, no inventing APIs). Replace the
language-specific sections (TS / Python / Go) with your language's
idioms — concrete examples: ownership conventions for Rust, null-safety
for Kotlin, value-vs-reference semantics for Swift.

Then state your language explicitly in `CLAUDE.md`. Without it the
model may slip back into TS/Python by default.

## Frontend frameworks beyond React

The prompts that touch components (`extract-component.md`,
`pr-review.md` examples, `e2e-scenarios.md`) use React. To adapt:

- **Vue:** components and composables instead of components and hooks.
- **Svelte:** `.svelte` files; the equivalent of `useState` is `$state`
  in Svelte 5.
- **Solid:** signals and effects instead of state and useEffect.
- **Angular:** components, services, and dependency injection — the
  refactor prompts apply, but the architectural assumptions differ
  more.

In every case, name your framework explicitly in `CLAUDE.md`. Models
default to React if not told otherwise.

## Backend frameworks

- **Node:** Hono / Fastify / Express / Nest — name yours.
- **Python:** FastAPI / Flask / Django — name yours.
- **Go:** stdlib `net/http` / Echo / Gin / Chi — name yours.
- **Other:** Rails, Phoenix, Laravel, Spring — works the same; name
  the framework and version.

For each, name your ORM / DB layer (Drizzle, Prisma, Kysely,
SQLAlchemy, GORM, Ent, ActiveRecord, etc.) so the assistant generates
queries in your dialect.

## A general rule

If a prompt or rule doesn't fit your stack, **edit it in place** rather
than fighting it at runtime. Five minutes of customisation now saves
five minutes per use, forever.
