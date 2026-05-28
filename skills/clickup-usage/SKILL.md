---
name: clickup-usage
description: Use when interacting with ClickUp for task management, project tracking, searching tasks, creating or updating work items, or any ClickUp workspace operations
---

# ClickUp Usage

Use the `clickup` CLI (in PATH via the plugin's `bin/`) to interact with ClickUp.
All commands output JSON. Errors go to stderr as `{"error": "..."}`.

## Command Reference

```text
clickup workspaces                                       list all workspaces
clickup spaces [--workspace-id ID]                       list spaces
clickup folders SPACE_ID                                 list folders in a space
clickup lists [--space-id ID | --folder-id ID]           list lists
clickup tasks search [QUERY] [OPTIONS]                   search/filter tasks
clickup tasks get TASK_ID                                get task details
clickup tasks create --list-id ID --name NAME [OPTIONS]  create a task
clickup tasks update TASK_ID [OPTIONS]                   update a task
clickup tasks delete TASK_ID                             delete a task
clickup comments list TASK_ID                            list task comments
clickup comments create TASK_ID --text TEXT              post a comment
```

### tasks search options

```text
--workspace-id  override CLICKUP_TEAM_ID
--list-id       scope to a specific list
--status        filter by status
--assignee      filter by user ID
--due-before    Unix ms timestamp upper bound
--due-after     Unix ms timestamp lower bound
--page          0-indexed page (ignored when QUERY is set)
```

Output: `{"tasks": [...], "has_more": bool}`.
With QUERY: fetches up to 500 tasks, filters by name substring client-side.
Without QUERY: one page from the server; check `has_more` and use `--page N` to paginate.

### tasks create / update options

```text
--description / -d    task description
--status / -s         status string (e.g. "in progress")
--priority            1=urgent 2=high 3=normal 4=low
--assignees           comma-separated user IDs (create)
--add-assignees       comma-separated user IDs (update)
--remove-assignees    comma-separated user IDs (update)
--due-date            Unix ms timestamp
```

## Workflow

### Navigation pattern

Hierarchy: **Workspace > Space > Folder > List > Task**

When you need an ID (e.g. a list ID to create a task), walk the hierarchy:

```bash
clickup spaces                          # get space IDs
clickup folders SPACE_ID                # get folder IDs
clickup lists --folder-id FOLDER_ID     # get list IDs
```

### Search before creating

Always run `clickup tasks search "name"` before creating to avoid duplicates.

### List selection

Infer the target list from context — ask only when genuinely ambiguous:

- User named or hinted at a list, space, or folder → use it
- Task topic maps clearly to one list (e.g. "login bug" → `Bugs`) → use it
- Recent session activity established a list and this task fits → reuse it

Ask only when multiple lists are equally plausible with no signal to break the tie.

### Multi-workspace

`--workspace-id` overrides `CLICKUP_TEAM_ID` on any command that accepts it.
To discover all workspaces: `clickup workspaces`.

## Environment

- `CLICKUP_API_KEY` — personal API token (`pk_...`) from ClickUp Settings > Apps
- `CLICKUP_TEAM_ID` — default workspace ID (first number in your ClickUp URL)

Both can be set in `~/.claude/settings.json` under `"env"`.
