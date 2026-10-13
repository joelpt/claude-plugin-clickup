# ClickUp Plugin for Claude Code

A Claude Code plugin that connects Claude to ClickUp.
It registers the [`clickup-mcp`](https://github.com/joelpt/clickup-mcp) MCP server, a direct [ClickUp REST API v2](https://clickup.com/api/) wrapper with no third-party service and no license key, and ships a skill that teaches Claude to drive its `mcp__clickup__*` tools.

## Features

90 tools, one per ClickUp operation, covering:

- Tasks, subtasks, search (including closed tasks), moves, dependencies, and links
- Workspaces, spaces, folders, and lists
- Comments, checklists, tags, and custom fields (including dropdowns)
- Time tracking, goals, views, and webhooks
- Multi-workspace support: every workspace your token can reach is discoverable

Each tool is gateable on its own, so you can allow reads and require confirmation for writes and deletes.

### Why not ClickUp's official MCP server?

ClickUp hosts its own server, which uses browser OAuth and, at the time of writing, documents daily call limits on lower plans and gaps in API coverage.
This plugin is the alternative when you want a personal API token that works headlessly (bots, CI, cron), no MCP-side call cap, and broader coverage.
It is an independent project and is not affiliated with ClickUp.
See [the server README](https://github.com/joelpt/clickup-mcp#why-this-server) for details.

## Prerequisites

1. **ClickUp API token** — generate a personal token (`pk_...`) in [ClickUp Settings > Apps](https://app.clickup.com/settings/apps).
   It has the full access of your ClickUp user, so treat it like a password.
2. **Team ID** (optional default workspace) — open ClickUp in your browser.
   The URL looks like `https://app.clickup.com/1234567/...`, and the first number is your Team ID.
3. **`uv`** — the server runs through `uvx` straight from GitHub.
   Install with `brew install uv` or `curl -LsSf https://astral.sh/uv/install.sh | sh`.

## Setup

Keep the token in a file readable only by you, and point the plugin at it:

```bash
install -m 600 /dev/null ~/.config/clickup-api-key
$EDITOR ~/.config/clickup-api-key   # paste the pk_... token
```

Then set these in the environment Claude Code starts from, for example in `~/.claude/settings.json`:

```json
{
  "env": {
    "CLICKUP_API_KEY_FILE": "/Users/you/.config/clickup-api-key",
    "CLICKUP_TEAM_ID": "1234567"
  }
}
```

`CLICKUP_API_KEY` also works if you prefer to pass the token directly, and it takes precedence when both are set.
Restart Claude Code after changing the environment.

## Usage

Describe what you want in plain language:

- "Search for tasks assigned to me in the Engineering space"
- "Create a task in the Sprint Backlog list called 'Fix login bug'"
- "Update task abc123 status to 'In Progress'"
- "Show me all tasks due this week"
- "Add a comment to task abc123: 'Fixed in PR #42'"

## How it works

The plugin's `.mcp.json` registers the `clickup` MCP server, launched on demand with `uvx --from git+https://github.com/joelpt/clickup-mcp@<commit-sha> clickup-mcp`.
The pin is an immutable full commit SHA: the commit that a CalVer release tag (for example `v2026.10.13.1`) points to.
That SHA matches the plugin's own version line, so the two signal a compatible pair, while the SHA keeps the reference tamper-proof because a tag could be force-moved.
`uvx` fetches and caches the server and its dependencies on first run.

## Security

Review the permissions you grant: the server exposes delete operations and webhook creation.
Report vulnerabilities as described in the [server's SECURITY.md](https://github.com/joelpt/clickup-mcp/blob/main/SECURITY.md).

## License

MIT
