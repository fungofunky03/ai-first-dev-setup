---
title: "Generate a README from a codebase"
category: docs
when_to_use: "When you need a first-pass README for an undocumented repo, or when an existing README has drifted from reality"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Generates a README that mirrors what the code actually does, not what the
maintainer intended it to do. The trick is to read the repo first
(`package.json`, entry points, top-level directories) and ground every
claim in a file path. Without that grounding, models produce plausible
READMEs that describe a different project.

The prompt also explicitly forbids marketing language. README readers are
developers evaluating whether to use the project; they want capability and
constraints, not "fast, lightweight, blazing".

## Prompt

```
Generate a README.md for the repository in this directory. Read these
files first (in order) and ground every claim in the README in something
you saw:

  1. `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` —
     dependencies, scripts, entry point.
  2. The top-level directory listing.
  3. Any existing `README*`, `CONTRIBUTING*`, `docs/*`.
  4. The main entry file (often `src/index.*`, `cmd/*`, or `main.*`).
  5. 2-3 representative source files (largest non-test files).

Structure (use these section headers verbatim):

  # <Project name>
  > <One-sentence elevator pitch — what it does, who it is for>

  ## What it is
  2-3 short paragraphs. Concrete. Cite at least one file path.

  ## Status
  One line: production / beta / experimental / abandoned. Justify from
  evidence (test coverage, last commit date, version, presence of CI).

  ## Quick start
  Copy-pasteable commands. Show install, configure (if needed), run, test.
  If the repo has a `.env.example`, list the required env vars.

  ## How it works
  A short explanation of the architecture in plain prose, plus a mermaid
  diagram if the architecture has more than 3 moving parts.

  ## Project layout
  A directory tree with one-line annotations on each top-level item.

  ## Configuration
  Every env var or config option the user can set. Table format: name,
  default, required (y/n), description.

  ## Contributing
  One paragraph + link to `CONTRIBUTING.md` if it exists, otherwise a
  short "open an issue first" note.

  ## License
  From `LICENSE` file.

Constraints:
- No marketing words: "fast", "blazing", "robust", "lightweight",
  "powerful", "seamless", "modern". Use specifics instead (latency
  number, dependency count, etc.) or remove.
- No emoji unless the repo already uses them (check existing markdown).
- No badges unless you can verify the badge URL from CI config.
- If a section would be empty, omit it. Do not write "TBD" or "Coming
  soon".
- Quote file paths in backticks. Cite line numbers when relevant.

After generation, list:
- Files you read.
- Claims you could not verify from the code (and removed).
```

## Example

**Input:** The current `ai-first-dev-setup` repo.

**Expected output (excerpt):**

> # ai-first-dev-setup
> > Curated boilerplate for an AI-first development environment — rules,
> > prompts, workflows, MCP configs. Set up in under 10 minutes.
>
> ## What it is
>
> `ai-first-dev-setup` is a documentation-only repository (no runtime
> code, no build step). It collects opinionated configuration files —
> `.cursorrules`, `CLAUDE.md`, `agent.md` — alongside 14 production-ready
> prompts (`prompts/`) and 4 end-to-end workflows (`workflows/`) that
> developers copy into their own projects.
>
> The library is multi-language: rules and prompts cover TypeScript /
> JavaScript, Python, and Go, with TypeScript / React used for the
> concrete examples.
>
> ## Status
>
> v1.0.0, public. MIT-licensed. Maintained by Learning-Heroes / HeroLabs.
> No automated CI (it is a prose repository); content is reviewed by
> Sendoa (frontend) and Rafa (backend) before each release.
>
> ## Quick start
>
> ```bash
> git clone https://github.com/Learning-Heroes/ai-first-dev-setup.git
> cd ai-first-dev-setup
>
> # Copy what you need into your own project:
> cp .cursorrules <your-project>/.cursorrules
> cp CLAUDE.md <your-project>/CLAUDE.md
> cp mcp/claude-desktop.example.json <your-mcp-config-location>
> ```
>
> No install, no build. Read `docs/getting-started.md` for the 10-minute
> walkthrough.
>
> [...rest of sections omitted...]
>
> Files read:
> - `package.json` — not present (prose repo, confirmed)
> - top-level `ls` — 8 directories, 5 files
> - `README.md` — existing version, drafted from RDD
> - `.cursorrules`, `CLAUDE.md`, `agent.md` — config grounding
> - `prompts/code-review/pr-review.md`, `workflows/feature-development.md` —
>   sample content
>
> Removed claims:
> - "Used by 1000+ developers" — could not verify from repo state.

## Personalisation tips

- **Library vs application:** for libraries, add an "API" section between
  "How it works" and "Project layout" with a 3-5 line example for the
  primary export. Append "Generate API examples from the public exports
  in `src/index.*`".
- **Polyglot repos:** name the language for each script
  ("(npm)", "(make)", "(just)") in the quick start to avoid confusion.
- **Heavy CLI:** if the repo is a CLI, add a "Commands" section. The
  prompt will discover them from the entry file's command parser.
- **Strict tone:** for very dry technical projects, append "Tone: terse,
  reference-manual style. No second-person ('you')."
- **Internal repos:** add "Audience: existing team members, not external
  users. Skip the elevator pitch." to make the README more reference-like.
