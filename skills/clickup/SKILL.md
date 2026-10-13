---
name: clickup
description: ClickUp task management & workspace
---

# ClickUp

This plugin registers the `clickup` MCP server (see `.mcp.json`). Interact with ClickUp
through its `mcp__clickup__*` tools — never shell out to the REST API directly.

## Tools

The server exposes 81 tools, each named `mcp__clickup__<tool>`.
Core ones, with their main arguments:

```text
mcp__clickup__list_workspaces                         list all workspaces (teams)
mcp__clickup__list_spaces      [workspace_id|workspace_name]
mcp__clickup__list_folders     space_id
mcp__clickup__list_lists       [space_id|folder_id]
mcp__clickup__search_tasks     [query] [filters…]     search/filter tasks
mcp__clickup__get_task         task_id
mcp__clickup__create_task      list_id name [parent] [opts…]   parent makes a subtask
mcp__clickup__update_task      task_id [opts…]
mcp__clickup__move_task        task_id list_id
mcp__clickup__delete_task      task_id
mcp__clickup__list_comments    task_id
mcp__clickup__add_comment      task_id text [notify_all]
```

The rest, by domain (use the tool name as listed, with the `mcp__clickup__` prefix):

- **Spaces, folders, lists** — `get_/create_/update_/delete_` for each, plus `create_folderless_list`, `get_list_members`
- **Tasks** — `get_task_members`, `get_task_time_in_status`, `add_dependency`, `remove_dependency`, `add_task_link`, `remove_task_link`
- **Checklists** — `create_/update_/delete_checklist` and `create_/update_/delete_checklist_item`
- **Comments** — `list_list_comments`, `create_list_comment`, `update_comment`, `delete_comment`
- **Custom fields** — `list_custom_fields`, `set_custom_field_value`, `remove_custom_field_value`
- **Tags** — `get_space_tags`, `create_/update_/delete_space_tag`, `add_task_tag`, `remove_task_tag`
- **Goals** — `list_goals`, `get_/create_/update_/delete_goal`, `create_/update_/delete_key_result`
- **Views** — `list_workspace_views`, `list_space_views`, `list_folder_views`, `list_list_views`, `get_view`, `get_view_tasks`, `create_/update_/delete_view`
- **Webhooks** — `list_webhooks`, `create_/update_/delete_webhook`
- **Time tracking** — `get_time_entries`, `get_running_time_entry`, `create_/update_/delete_time_entry`, `start_timer`, `stop_timer`
- **People** — `list_members`, `get_authorized_user`

Deletes and webhook creation are destructive or outward-facing: confirm with the user before calling them.

`search_tasks` hides closed tasks by default; pass `include_closed=true` for a full audit.
For a dropdown custom field, pass the option's `id` to `set_custom_field_value`.

All tools return JSON text. Errors surface as MCP tool errors with a message.

### Task fields (create / update)

`description`, `status` (e.g. "in progress"), `priority` (1=urgent 2=high 3=normal
4=low), `assignees` / `add_assignees` / `remove_assignees` (lists of numeric user ids),
`due_date` (Unix ms timestamp). `search_tasks` filters: `status`, `assignee`,
`due_before`, `due_after`, `list_id`, `page`; with `query` it filters task names
client-side over up to 500 tasks and returns `{"tasks": [...], "has_more": bool}`.

## Presenting results

Default to a **Markdown table** when returning more than one item — tasks, spaces, lists,
folders, comments. One row per item; pick columns that match the request (commonly name,
status, priority, assignee, due date, id). A table makes the cross-item comparison the user
usually wants scannable at a glance.

Deviate when another shape is the better fit:

- The user asked for or implied a different format (prose summary, just the ids, a checklist) → honor it.
- A single item, or a genuinely narrative answer → prose, not a one-row table.
- Items the user will act on in order (steps, a backlog to work top-down) → a numbered list;
  a handful of loosely-related one-liners → a bulleted list.

Never dump raw tool JSON at the user — it's the wire format, not a presentation.

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
