# Getting started

A 10-minute walkthrough from "I cloned the repo" to "I have an AI-first
dev environment configured for one of my own projects". If you've used
Cursor or Claude Desktop before, it's closer to 5 minutes.

## Prerequisites

- An IDE that supports AI: Cursor, Windsurf, VS Code with Claude Code,
  Claude Desktop, or another MCP-capable host.
- A project of yours to configure (any language; examples target
  TypeScript / Python / Go).
- Optional: a GitHub Personal Access Token (fine-grained, scoped to the
  repos you want the AI to see) if you want the GitHub MCP.

## Step 1 — Pick what you need

The repo is a buffet, not a meal. You don't copy everything. The
typical first-time setup uses:

- `.cursorrules` (or `.windsurfrules`)
- `CLAUDE.md` (filled with your project context — use the template)
- One MCP config from `mcp/`
- One workflow from `workflows/`
- A handful of prompts you'll actually use

Skip the rest until you need it.

## Step 2 — Copy the rules

Copy the rules file your IDE reads:

```bash
# For Cursor users
cp ai-first-dev-setup/.cursorrules <your-project>/.cursorrules

# For Windsurf users
cp ai-first-dev-setup/.windsurfrules <your-project>/.windsurfrules
```

The rules cover TypeScript, Python, and Go. If your project is in
another language, the structural rules (plan-confirm-execute, no silent
catches, anti-patterns) still apply — keep the file, ignore the
language-specific clauses.

## Step 3 — Create your CLAUDE.md

```bash
cp ai-first-dev-setup/templates/CLAUDE.md.template <your-project>/CLAUDE.md
```

Open it and fill in:

- **Project context:** what is this project, who is it for.
- **Tech stack:** languages, frameworks, services, versions.
- **Conventions:** the unwritten rules a new hire would have to ask
  about (file naming, where types live, branch naming).
- **Do / Don't:** behaviours you want and behaviours you've already
  caught the AI getting wrong.
- **Key files:** the 5-10 files an assistant would need to know about
  to navigate efficiently.

The whole file should be ≤ 250 lines. Brevity matters — every line is
prepended to every conversation.

## Step 4 — Set up MCP servers

Pick the config that matches your IDE:

```bash
# Claude Desktop
cp ai-first-dev-setup/mcp/claude-desktop.example.json \
  ~/Library/Application\ Support/Claude/claude_desktop_config.json
# Then edit to replace <PLACEHOLDERS>

# Cursor
cp ai-first-dev-setup/mcp/cursor.example.json ~/.cursor/mcp.json
# Then edit to replace <PLACEHOLDERS>
```

At minimum, enable **sequential-thinking** (no setup needed) and the
**filesystem** server pointed at your project. Add GitHub later if you
need cross-repo context.

Restart your IDE / Claude Desktop after editing.

## Step 5 — Try one workflow end-to-end

Pick a small feature you'd otherwise build today. Open it as your
first AI-assisted task using `workflows/feature-development.md`:

1. Write the "What / Why / Where / Constraints / Done when" message.
2. Ask for a plan.
3. Approve or amend.
4. Implement in slices.
5. Verify manually.

The first time will feel slow — that's the point. By the third feature
the cadence is automatic and faster than your previous workflow.

## Step 6 — Adopt 1-2 prompts

Don't try to use all 14 prompts. Pick two that match your most common
friction:

- Spending too long on code review? → `prompts/code-review/pr-review.md`.
- Spending too long debugging? → `prompts/debug/investigate-bug.md`.
- Writing tests grudgingly? → `prompts/tests/unit-tests-from-fn.md`.
- Generating docs from scratch repeatedly? →
  `prompts/docs/readme-generator.md`.

Bookmark them. Use them. Add more once those two are habitual.

## Where to go next

- Read `docs/philosophy.md` to understand why the rules and prompts
  are shaped the way they are. Reject the parts you disagree with —
  but reject them deliberately, after reading.
- Read `docs/customization.md` for per-stack adaptation (TS / Python /
  Go-specific tweaks).
- Read the workflow most relevant to your current work
  (`workflows/spec-driven-development.md` for new features,
  `workflows/onboarding-new-codebase.md` if you're new to a repo).
- Contribute back: see `CONTRIBUTING.md` to propose new prompts or
  fixes.

## Common first-day issues

- **"My AI is ignoring `CLAUDE.md`."** Check it's at the project root,
  not in a subdirectory. Restart the IDE / Claude Desktop after creating
  it.
- **"My AI keeps proposing files that don't exist."** The rules file is
  there but the model isn't reading it. Verify it's at the right path
  (`.cursorrules` not `.cursor-rules`) and that the IDE is configured
  to read project rules.
- **"MCP server isn't starting."** Check the command path in the config
  — `npx` must be on PATH for the IDE's environment. Try the full path
  to `npx` if not.
- **"The AI is too verbose."** Add a line to `CLAUDE.md` under "Do":
  "Default to terse responses; only expand when the user asks for more
  detail." Or add it to the rules file directly.
