# Onboarding to a new codebase with AI

A reading-order recipe for when you've just been dropped into an
unfamiliar repo and need to be productive in hours, not weeks. The
workflow trades passive code-reading for active conversations with an AI
assistant that can read alongside you.

## When to apply

- Joining a new team or project.
- Inheriting a repo from someone who left.
- Auditing a repo before deciding to use, fork, or buy it.
- Returning to a project you have not touched in months.

This is a **checklist**, not a strict sequence — most steps run in
parallel as your understanding grows.

## Required tools

- An AI assistant with filesystem access (Cursor opens the workspace
  directly; Claude Desktop needs the filesystem MCP pointed at the repo).
- Recommended MCPs: filesystem, github (for issue/PR archaeology).
- Local clone of the repo plus the ability to run its tests.

## The checklist

### Hour 1 — orient

- [ ] **Read the README.** If it is missing or stale, run
      `prompts/docs/readme-generator.md` against the repo and read *that*.
- [ ] **Skim the top-level directory.** Ask the AI:
      > "List the top-level directories and files in this repo. For each,
      > guess the purpose based on the name and contents."
      Cross-check against the README's stated layout. Mismatches are
      signal — the codebase has drifted from the docs.
- [ ] **Identify the entry point(s).** Where does control start?
      `src/index.*`, `cmd/*`, `main.*`, `app/`. Ask the AI to trace from
      entry to the first significant business logic.
- [ ] **Identify the data model.** Find the `models/`, `schema.sql`,
      `prisma/schema.prisma`, `pyproject.toml`, ORM annotations,
      protobufs. Ask the AI:
      > "Summarise the data model: entities, relationships,
      > authoritative source for the schema."

### Hour 2 — discover the conventions

- [ ] **Read the most-changed files** (use
      `git log --pretty=format: --name-only | sort | uniq -c | sort -rg`
      and take the top 5). High-churn files are usually high-importance.
- [ ] **Find the testing strategy.** Where do tests live? Unit vs
      integration vs E2E ratios? Ask the AI:
      > "Show me 3 representative tests of each type (unit, integration,
      > E2E) and explain the project's testing conventions."
- [ ] **Find the linting/formatting config.** `.eslintrc`,
      `pyproject.toml`, `.editorconfig`, `Makefile`. These encode team
      decisions. Read them.
- [ ] **Identify any `CONTRIBUTING.md` or implicit conventions** (commit
      message format, branch naming, PR template).

### Hour 3 — find the danger zones

- [ ] **Run `git log --since="3.months.ago" --oneline | head -50`** and
      ask the AI:
      > "Looking at recent commits, what areas of this codebase have
      > been actively maintained, and what looks abandoned?"
- [ ] **Find the issue tracker hotspots.** Open recent issues/PRs in
      GitHub. Ask the AI (via the GitHub MCP):
      > "What recurring themes appear in the last 20 issues?"
- [ ] **Look at the TODOs and FIXMEs.**
      `grep -rn "TODO\|FIXME\|XXX\|HACK" src/`. The AI is good at
      clustering these into themes.
- [ ] **Find the runtime gotchas:** any custom build steps, env-var
      requirements, services that must be running locally. Run the
      project end-to-end at least once.

### Hour 4 — write your own map

- [ ] **Draft a `CLAUDE.md`** for the repo if it does not have one.
      Use `templates/CLAUDE.md.template` as a starting point. Iterate
      with the AI:
      > "I'm drafting a CLAUDE.md for this repo. Critique it for
      > accuracy and completeness. What did I get wrong, what did I miss?"
- [ ] **Write yourself a 1-page mental model** — entry points, key data
      types, where business logic lives, where to *avoid* touching
      first. Save it in your own notes, not in the repo (it will be
      wrong in places and that's fine).

### Beyond hour 4 — earn it by changing it

The fastest way to consolidate understanding is to ship a small,
real change. Look for:

- A `good-first-issue` label in the tracker.
- A TODO you can clear.
- A test you can add for a function with thin coverage (use
  `prompts/tests/edge-case-finder.md`).

Resist the temptation to start with a refactor. Refactor only after
you've shipped at least one bug fix or feature; until then you don't
know enough to refactor without breaking invariants.

## Anti-patterns to avoid

- **Reading top-to-bottom.** Codebases are not novels. Read by
  dependency from entry points, or by feature from user surface.
- **Asking the AI to "explain this codebase".** Too broad; you get a
  generic summary. Ask narrow questions tied to concrete files.
- **Trusting the AI's first answer.** Cross-check against the actual
  files. AI assistants confabulate confidently on unfamiliar repos.
- **Reading every file.** Use the most-changed files heuristic; the
  long tail is rarely worth the time.
- **Updating documentation before you understand the code.** Your
  changes will be wrong. Wait until you've shipped one change.

## What "done" looks like

You can:

1. Run the tests and explain what each suite is checking.
2. Trace a user request from entry to exit, naming the key files.
3. Pick a random PR and predict whether it should be approved.
4. Make a one-paragraph case for why one specific area of the code
   should be refactored — and a counter-argument for leaving it alone.

If you cannot do (4), you do not understand the code yet. Keep reading.
