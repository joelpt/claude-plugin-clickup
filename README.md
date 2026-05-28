# ClickUp Plugin for Claude Code

A Claude Code plugin that integrates with ClickUp for project management.
It registers the [`clickup-mcp`](https://github.com/joelpt/clickup-mcp) MCP server (a direct
[ClickUp REST API v2](https://clickup.com/api/) wrapper — no third-party service, no license key)
and ships a skill that drives its `mcp__clickup__*` tools.

## Features

- Task management (create, update, search, delete)
- Workspace navigation (spaces, folders, lists)
- Comments
- Multi-workspace support (auto-discover all workspaces via API key)

## Prerequisites

1. **ClickUp API Key** — Go to [ClickUp Settings > Apps](https://app.clickup.com/settings/apps) and generate a personal API token (`pk_...`).

2. **ClickUp Team ID** — Open ClickUp in your browser.
The URL looks like `https://app.clickup.com/1234567/...` — the first number (`1234567`) is your Team ID.

3. **`uv`** — The MCP server runs via [uv](https://docs.astral.sh/uv/)/`uvx` straight from GitHub (zero install).
Install with `brew install uv` or `curl -LsSf https://astral.sh/uv/install.sh | sh`.

## Setup

Add to `~/.claude/settings.json`:

```json
{
  "env": {
    "CLICKUP_API_KEY": "pk_your_token_here",
    "CLICKUP_TEAM_ID": "your_team_id_here"
  }
}
```

Then restart Claude Code.

## Usage

Once installed, describe your ClickUp task naturally:

- "Search for tasks assigned to me in the Engineering space"
- "Create a task in the Sprint Backlog list called 'Fix login bug'"
- "Update task abc123 status to 'In Progress'"
- "Show me all tasks due this week"
- "Add a comment to task abc123: 'Fixed in PR #42'"

## How it works

The plugin's `.mcp.json` registers the `clickup` MCP server, launched on demand via
`uvx --from git+https://github.com/joelpt/clickup-mcp@<pinned-sha> clickup-mcp`. Each ClickUp
operation is its own `mcp__clickup__*` tool, so an MCP host can permit or gate them individually
(e.g. allow reads, gate writes). `uvx` fetches and caches the server (and its deps) on first run.

## License

MIT
