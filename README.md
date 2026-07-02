# ai-first-dev-setup

> Configure your AI-first dev environment in 10 minutes.

Curated rules, prompts, workflows, and MCP configs for developers who
want their AI assistant to actually understand the project before it
writes a line of code. Copy what you need into your own repo. No
install, no build, no runtime — this is configuration and prose.

## Why this exists

AI-assisted development is now the default. Most teams adopt it
accidentally: pasting code into a chat, accepting whatever comes back,
shipping it. They are slightly faster than before, and substantially
less reliable than they should be.

The leverage is in *configuration*: an assistant that knows your
project conventions, follows your team's rules, and produces work in
slices you can actually review. That is what this repo provides.

It is **opinionated**. The opinions are documented in
[`docs/philosophy.md`](./docs/philosophy.md). Read it before disagreeing.

## Quick start

```bash
git clone https://github.com/Learning-Heroes/ai-first-dev-setup.git
cd ai-first-dev-setup

# Copy the rules into your own project
cp .cursorrules <your-project>/.cursorrules         # for Cursor
cp .windsurfrules <your-project>/.windsurfrules     # for Windsurf

# Drop in a project-specific CLAUDE.md
cp templates/CLAUDE.md.template <your-project>/CLAUDE.md
# Then edit it: fill in tech stack, conventions, key files.

# Wire up MCPs (Claude Desktop example)
cp mcp/claude-desktop.example.json \
  ~/Library/Application\ Support/Claude/claude_desktop_config.json
# Edit to replace <PLACEHOLDERS>. Restart Claude Desktop.
```

For the full walkthrough, see
[`docs/getting-started.md`](./docs/getting-started.md).

## What's inside

| Path                   | Purpose                                                                 |
| ---------------------- | ------------------------------------------------------------------------ |
| `.cursorrules`         | Strict, prioritised rules for AI agents (multi-language: TS, Python, Go) |
| `.windsurfrules`       | Mirror of `.cursorrules` for Windsurf users                              |
| `CLAUDE.md`            | Project-level context for Claude Desktop / Code (this repo's own context) |
| `agent.md`             | Universal multi-IDE agent template (Claude Desktop, Cursor, Windsurf, Cline) |
| `mcp/`                 | Example MCP server configs with sensible defaults                       |
| `prompts/`             | 17 production-ready prompts across 5 categories                          |
| `workflows/`           | 4 end-to-end recipes (spec-driven, onboarding, feature dev, PR flow)    |
| `templates/`           | Starting points: `CLAUDE.md`, agent definition, feature spec, ADR        |
| `docs/`                | Philosophy (EN + ES), getting started, customization                     |

### Prompt categories

- **`code-review/`** — PR review, security audit, performance review
- **`refactor/`** — extract component, reduce complexity, migrate pattern
- **`debug/`** — investigate a bug, root cause analysis, triage classifier
- **`docs/`** — README generator, API docs, changelog from commits, summarize module
- **`tests/`** — unit tests from function, E2E scenarios, edge-case finder, LLM-as-judge eval

Each prompt has YAML frontmatter (`title`, `category`, `when_to_use`,
`tested_with`, `version`), a concrete TypeScript / React example, and
personalisation tips.

### Workflows

- **`spec-driven-development.md`** — for features that justify a spec
- **`feature-development.md`** — lightweight version for small changes
- **`onboarding-new-codebase.md`** — get productive in a new repo in
  hours
- **`pr-workflow.md`** — author and review PRs with AI

## How to customize per stack

The defaults target TypeScript / React. For Python, Go, and other
languages, see
[`docs/customization.md`](./docs/customization.md) — it lists which
rules and prompts need per-stack tweaks and which transfer unchanged.

## Models tested with

Prompts in this repo are tested against:

- Claude Opus 4.7
- Claude Sonnet 4.6
- GPT-5

If a prompt requires a model-specific feature, the `tested_with`
frontmatter narrows the list.

## Contributing

This is a curated library, not an open dumping ground. Quality bar:
every prompt or workflow ships with a concrete example the author has
personally run.

See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for the prompt-submission
template, frontmatter checklist, and style guide. Open an issue first
for substantial additions.

## License

[MIT](./LICENSE). Use it, modify it, ship it.

## Maintained by

[Learning-Heroes / HeroLabs](https://github.com/Learning-Heroes).

For the Spanish version of the philosophy, see
[`docs/philosophy.es.md`](./docs/philosophy.es.md).
