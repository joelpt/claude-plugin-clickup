---
name: clickup
description: Use when interacting with ClickUp for task management, project tracking, searching tasks, creating or updating work items, or any ClickUp workspace operations
---

# ClickUp

This plugin registers the `clickup` MCP server (see `.mcp.json`). Interact with ClickUp
through its `mcp__clickup__*` tools — never shell out to the REST API directly.

## Tools

```text
mcp__clickup__list_workspaces                         list all workspaces (teams)
mcp__clickup__list_spaces      [workspace_id|workspace_name]
mcp__clickup__list_folders     space_id
mcp__clickup__list_lists       [space_id|folder_id]
mcp__clickup__search_tasks     [query] [filters…]     search/filter tasks
mcp__clickup__get_task         task_id
mcp__clickup__create_task      list_id name [opts…]
mcp__clickup__update_task      task_id [opts…]
mcp__clickup__delete_task      task_id
mcp__clickup__list_comments    task_id
mcp__clickup__add_comment      task_id text [notify_all]
```

All tools return JSON text. Errors surface as MCP tool errors with a message.

### Task fields (create / update)

`description`, `status` (e.g. "in progress"), `priority` (1=urgent 2=high 3=normal
4=low), `assignees` / `add_assignees` / `remove_assignees` (lists of numeric user ids),
`due_date` (Unix ms timestamp). `search_tasks` filters: `status`, `assignee`,
`due_before`, `due_after`, `list_id`, `page`; with `query` it filters task names
client-side over up to 500 tasks and returns `{"tasks": [...], "has_more": bool}`.

## Workflow

### Navigation

Hierarchy: **Workspace > Space > Folder > List > Task**. To find an id (e.g. a list id
to create a task in), walk down: `list_spaces` → `list_folders` → `list_lists`.

### Choosing a workspace

`list_spaces` / `list_lists` / `search_tasks` / `create_task` default to the configured
workspace. To target another, pass `workspace_name` (resolved by name server-side) or
`workspace_id` — e.g. "add a task in the VeggieCo workspace" → pass `workspace_name="VeggieCo"`.

### Search before creating

Run `search_tasks` with the task name before `create_task` to avoid duplicates.

### List selection

Infer the target list from context — ask only when genuinely ambiguous:

- User named or hinted at a list, space, or folder → use it.
- Task topic maps clearly to one list (e.g. "login bug" → `Bugs`) → use it.
- Recent activity established a list and this task fits → reuse it.

## Configuration

- `CLICKUP_API_KEY` — personal API token (`pk_...`) from ClickUp Settings > Apps.
- `CLICKUP_TEAM_ID` — default workspace id (first number in your ClickUp URL).

Set both in your environment (the `.mcp.json` passes them through to the server).
