# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-05-26

Initial public release.

### Added

- Root configuration files: `.cursorrules`, `.windsurfrules`, `CLAUDE.md`,
  `agent.md`.
- MCP server configs: `mcp/claude-desktop.example.json`,
  `mcp/cursor.example.json`, and `mcp/README.md` covering filesystem,
  GitHub, sequential-thinking, and memory servers.
- Prompt library with 14 production-ready prompts across 5 categories:
  - `code-review/` — PR review, security audit, performance review.
  - `refactor/` — extract component, reduce complexity, migrate pattern.
  - `debug/` — investigate a bug, root-cause analysis.
  - `docs/` — README generator, API docs, changelog from commits.
  - `tests/` — unit tests from function, E2E scenarios, edge-case finder.
- 4 end-to-end workflows: spec-driven development, onboarding to a new
  codebase, lightweight feature development, PR workflow.
- Templates: `PROJECT_AGENT.md.template`, `feature-spec.md.template`,
  `adr.md.template`, `CLAUDE.md.template`.
- Documentation: `docs/philosophy.md` (+ Spanish translation
  `philosophy.es.md`), `docs/getting-started.md`, `docs/customization.md`.
- `CONTRIBUTING.md` with prompt-submission template, frontmatter
  checklist, and style guide.
- MIT `LICENSE`.

[1.0.0]: https://github.com/Learning-Heroes/ai-first-dev-setup/releases/tag/v1.0.0
