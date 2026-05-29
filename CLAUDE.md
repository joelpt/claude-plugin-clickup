# claude-plugin-clickup — project notes

This plugin registers the `clickup` MCP server (engine lives at `~/code/clickup-mcp`, GitHub `joelpt/clickup-mcp`) via `.mcp.json` and ships the `clickup` skill.
Plugin and server are a **matched pair**: the plugin is a thin registration + skill layer over the server, so server changes almost always entail a `.mcp.json` repin here.

## Versioning & the server pin fit together

- `.mcp.json` pins the server by **immutable full commit SHA** — the commit a CalVer release tag (`vYYYY.MM.DD.N`) points to — never a mutable tag or branch.
  A tag can be force-moved, which would let `uvx` silently fetch different code (supply-chain regression); the SHA can't. Document the matching tag in prose, but pin the SHA.
  The commit must already be pushed upstream or `uvx` resolution fails for every consumer.
- Keep this plugin's `version` (`.claude-plugin/plugin.json`, CalVer per `~/.claude/rules/claude-plugins.md`) **equal to the pinned server version** whenever a server release drives the bump — identical CalVer across both makes compatibility obvious to consumers.
- The plugin still bumps independently (per-commit CalVer counter) for plugin-only changes, so the two won't *always* be equal — but on a coordinated server release, sync them.

## When the server is updated

1. Release the server first (tag + push) — see `~/code/clickup-mcp/CLAUDE.md`.
2. Repin `.mcp.json` to the new release's **commit SHA** (the one its tag points to) and bump `.claude-plugin/plugin.json` to match the CalVer, then run the usual marketplace-sync (`claude-plugins.md`).
3. Update `~/code/cclaw`'s pin too (`CLICKUP_MCP_REF` in `bot/Dockerfile` + the fallback in `bot/entrypoint.sh`) so the bot fleet tracks the same server revision; cclaw needs its own commit + image rebuild.
