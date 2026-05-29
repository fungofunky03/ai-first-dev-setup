# MCP server configs

Model Context Protocol (MCP) servers extend your AI assistant with read/write
access to external systems — filesystems, GitHub, databases, anything that
has an MCP implementation. This directory contains starting configs for the
two most common hosts (Claude Desktop and Cursor) with the servers we
recommend by default.

## What's in here

| File                            | For                                 |
| ------------------------------- | ----------------------------------- |
| `claude-desktop.example.json`   | Claude Desktop (macOS, Windows)     |
| `cursor.example.json`           | Cursor IDE                          |

## How to use

1. Pick the example that matches your host.
2. Copy it to the location your host expects:
   - **Claude Desktop (macOS):** `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Claude Desktop (Windows):** `%APPDATA%\Claude\claude_desktop_config.json`
   - **Cursor (global):** `~/.cursor/mcp.json`
   - **Cursor (per-project):** `<project-root>/.cursor/mcp.json`
3. Replace every `<PLACEHOLDER>` with a real value.
4. Restart the host (Claude Desktop or Cursor).

## The four default servers

### Filesystem

Gives the model read access to a directory. Use the **most restrictive path
possible** — everything inside that path is fully readable to the model and
to whoever can see your conversation history.

For Claude Desktop, this is how you grant project access. For Cursor, the
workspace is already readable; only add Filesystem if you need access to a
sibling repo.

### GitHub

Lets the model read issues, PRs, and code across the repos your PAT can
see. Use a **fine-grained personal access token** scoped to the specific
repos and the minimum necessary permissions (typically: Contents read,
Issues read/write, Pull requests read/write).

Never use a classic token here. They are too broad and cannot be scoped per
repo.

### Sequential thinking

A local-only "scratchpad" server that helps the model break down complex
problems into ordered steps. No network, no secrets, no risk. Recommended
for everyone.

### Memory (optional)

Persists facts across conversations into a local SQLite database. Useful
for: project context that does not change ("we use Postgres 16"), user
preferences ("respond in English"), running notes.

Disable if you prefer every conversation to start fresh.

## Adding your own MCPs

The community catalogue is at
[github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers).
Common useful additions:

- **Postgres / SQLite** — read-only queries against a development database.
- **Puppeteer** — browser automation for scraping or screenshot capture.
- **Slack** — read/post in channels (use with care).
- **Linear / Jira** — pull ticket context into the conversation.

To add a server, append a new entry to the `mcpServers` object in your host
config. Each entry needs `command`, `args`, and optionally `env`.

## Security checklist before enabling any MCP

1. Does this MCP read secrets (env vars, files)? If yes, can you scope it
   tighter?
2. Does this MCP write? If yes, is the blast radius understood (one repo, a
   shared channel, a production database)?
3. Is the source of the MCP server trusted? Prefer official `@modelcontextprotocol/*`
   packages, then known maintainers, then individuals.
4. Are credentials in env vars rather than command-line args? (Args are
   visible in process listings.)
