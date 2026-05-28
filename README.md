# ClickUp Plugin for Claude Code

A Claude Code plugin that integrates with ClickUp for project management.
Calls the [ClickUp REST API v2](https://clickup.com/api/) directly — no third-party MCP server required.

## Features

- Task management (create, update, search, delete)
- Workspace navigation (spaces, folders, lists)
- Comments
- Multi-workspace support (auto-discover all workspaces via API key)

## Prerequisites

1. **ClickUp API Key** — Go to [ClickUp Settings > Apps](https://app.clickup.com/settings/apps) and generate a personal API token (`pk_...`).

2. **ClickUp Team ID** — Open ClickUp in your browser.
The URL looks like `https://app.clickup.com/1234567/...` — the first number (`1234567`) is your Team ID.

3. **`uv`** — The script uses [uv](https://docs.astral.sh/uv/) for zero-install dependency management.
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

The plugin adds a `clickup` CLI to your session PATH.
Claude calls it with the Bash tool — no MCP server subprocess, no npm, no license key.
The script auto-installs its Python dependencies (`httpx`, `typer`) via `uv` on first run and caches them.

## License

MIT
