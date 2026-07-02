# CLAUDE.md — ai-first-dev-setup

This file is read automatically by Claude Code and Claude Desktop when working
in this repository. It describes the project itself so the assistant has
project-level context without needing to be re-briefed each session.

If you are looking at this file because you want to add a `CLAUDE.md` to
*your own project*, use `templates/CLAUDE.md.template` instead — it is the
placeholder version.

## Project context

`ai-first-dev-setup` is an open-source boilerplate that developers clone to
configure an AI-first development environment in under 10 minutes. It ships
curated rules (`.cursorrules`, `.windsurfrules`), agent definitions
(`agent.md`, `CLAUDE.md`), MCP configs, a library of prompts, and end-to-end
workflows. It is not an interactive product — it is configuration and prose.

Distributed as a lead magnet for the "IA para Developers" program by
Learning-Heroes / HeroLabs. Public on GitHub under MIT, optimised for
organic discovery (stars/forks) rather than gated access.

## Repository structure

```
.
├── README.md                # Public entry point: hero, quick start, what's inside
├── CLAUDE.md                # This file — context for THIS repo
├── CONTRIBUTING.md          # How external contributors propose prompts/workflows
├── CHANGELOG.md             # Keep-a-Changelog format, one entry per release
├── LICENSE                  # MIT
├── .cursorrules             # Strict, prioritised agent rules (TS, Python, Go)
├── .windsurfrules           # Verbatim mirror of .cursorrules
├── agent.md                 # Universal multi-IDE agent template (placeholders)
├── docs/
│   ├── philosophy.md        # The *why* behind every choice (read before refactoring)
│   ├── philosophy.es.md     # Spanish translation of philosophy.md (only ES file)
│   ├── getting-started.md   # Full setup walkthrough
│   └── customization.md     # Per-stack tweaks (Python, Go, …) vs. what transfers
├── mcp/
│   ├── README.md
│   ├── claude-desktop.example.json
│   └── cursor.example.json
├── prompts/                 # 14 prompts across 5 categories (see prompts/README.md)
│   ├── README.md            # Prompt file schema + frontmatter spec
│   ├── code-review/         # pr-review, security-audit, performance-review
│   ├── refactor/            # extract-component, reduce-complexity, migrate-pattern
│   ├── debug/               # investigate-bug, root-cause-analysis
│   ├── docs/                # readme-generator, api-docs, changelog-from-commits
│   └── tests/               # unit-tests-from-fn, e2e-scenarios, edge-case-finder
├── workflows/               # 4 end-to-end recipes
│   ├── spec-driven-development.md
│   ├── feature-development.md
│   ├── onboarding-new-codebase.md
│   └── pr-workflow.md
└── templates/               # Starting points users copy into their own projects
    ├── CLAUDE.md.template
    ├── PROJECT_AGENT.md.template
    ├── adr.md.template
    └── feature-spec.md.template
```

## Tech stack

There is no runtime code in this repo. Files are:

- Markdown (`.md`) for documentation, prompts, workflows, templates.
- YAML frontmatter inside the Markdown for prompt metadata.
- JSON (`.json`) for MCP example configs.
- Plain text for rules files (`.cursorrules`, `.windsurfrules`).

No build, no compile, no tests to run. Quality control is human review by
the claustro (Sendoa for frontend-flavoured content, Rafa for backend, Iskren
for overall curation).

Prompts are tested against the models listed in each prompt's `tested_with`
frontmatter. As of the current release the baseline set is Claude Opus 4.7,
Claude Sonnet 4.6, and GPT-5. `tested_with` narrows this list when a prompt
depends on a model-specific feature.

## Development workflow

There is nothing to build or run, so "development" means editing prose and
config and getting it reviewed. Typical loop:

1. **Branch.** Work on a feature branch, never directly on `main`.
2. **Edit.** Add or change files following the conventions below. Keep changes
   scoped — one prompt, one workflow, or one doc per PR where possible.
3. **Self-check.** There are no automated tests; instead:
   - If you added/changed a prompt, run it against at least one model in its
     `tested_with` list and confirm the example still reproduces.
   - If you edited `.cursorrules`, mirror the exact change into `.windsurfrules`
     (they must stay byte-for-byte identical).
   - If you bumped a prompt's `version`, add a matching `CHANGELOG.md` entry.
   - If you edited `docs/philosophy.md`, update `docs/philosophy.es.md` to match.
4. **PR.** Open a pull request. See `CONTRIBUTING.md` for the submission
   template and frontmatter checklist. Substantial additions should start with
   an issue. Human review by the claustro is the merge gate.

## Conventions

- All file content is English. The single exception is `docs/philosophy.es.md`,
  which is the Spanish translation of `docs/philosophy.md`.
- Prompt files live under `prompts/<category>/<slug>.md` and always carry the
  YAML frontmatter described in `prompts/README.md`.
- Workflow files live under `workflows/<slug>.md` and include a mermaid
  diagram when the flow has more than 3 sequential steps.
- Templates use `<PLACEHOLDER_NAME>` (UPPER_SNAKE) for variables the reader
  is expected to replace. Each placeholder is documented inline the first
  time it appears.
- Examples in prompts use TypeScript / React unless the prompt is explicitly
  stack-agnostic (e.g. `docs/changelog-from-commits.md`).

## Do

- Keep the developer-native tone. Direct, technical, no marketing fluff.
- When adding a prompt, include a before/after concrete example, not just a
  description.
- When adding a workflow, state the entry condition ("when to apply") and the
  expected output of each step.
- Test every prompt against at least one of the models listed in
  `tested_with` before changing its version number.
- Cite file paths with line numbers when referencing source (`path:line`).

## Don't

- Don't add JavaScript or Python tooling unless the user asks. This is a
  prose repo; npm/pip add friction with no upside.
- Don't translate the README or other docs into Spanish beyond
  `philosophy.es.md`. The audience reads English.
- Don't link to internal Learning-Heroes URLs that are not public. Anything
  referenced in a public file must itself be public.
- Don't bump the version of a prompt without a corresponding `CHANGELOG.md`
  entry.
- Don't add MCPs to `mcp/*.example.json` that require paid API keys without
  marking them clearly as optional.

## Key files and their purpose

- `README.md` — public entry point. Hero + quick start + what's inside.
- `.cursorrules` / `.windsurfrules` — strict, prioritised rules an AI agent
  must follow inside a user's project. Multi-language (TypeScript, Python, Go).
  Mirror each other verbatim — edit both or neither.
- `agent.md` — universal multi-IDE agent template (Claude Desktop, Cursor,
  Windsurf, Cline). Placeholders for the user's project.
- `CLAUDE.md` — this file. Describes *this* repo.
- `templates/CLAUDE.md.template` — placeholder version users copy into their
  own projects.
- `mcp/*.example.json` — copy-paste MCP server configs.
- `prompts/<category>/*.md` — 14 production-ready prompts across 5 categories.
- `workflows/*.md` — 4 end-to-end recipes (spec-driven dev, onboarding,
  feature dev, PR workflow).
- `templates/*.template` — feature spec, ADR, agent definition starting
  points.
- `docs/philosophy.md` (+ `.es.md`) — the *why* behind every choice in the
  repo. Read this before refactoring anything structural.
- `docs/getting-started.md` — the full setup walkthrough the README links to.
- `docs/customization.md` — which rules/prompts need per-stack tweaks (Python,
  Go, …) and which transfer unchanged.
- `prompts/README.md` — the prompt file schema and frontmatter spec. Read
  before adding a prompt.
- `mcp/README.md` — how to use the example MCP configs.
- `CONTRIBUTING.md` — how external contributors propose new prompts.
- `CHANGELOG.md` — Keep-a-Changelog format; one entry per release.
