---
name: my-tasks
description: List all your assigned JIRA tasks in the current project
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
---

# My Tasks

Show all JIRA tickets currently assigned to the user in the configured project. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Story Points Field

Story points live in `customfield_10028` on this JIRA instance.

**Important:** This field is NOT returned by default by MCP tools. When calling `mcp__atlassian__jira_search` or `mcp__atlassian__jira_get_issue`, you **must** include it in the `fields` parameter:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,customfield_10028"
```

## Behavior

### 1. Fetch Assigned Tickets

Use `mcp__atlassian__jira_search` with JQL:
```
project = HYPERFLEET AND assignee = currentUser() ORDER BY updated DESC
```

### 2. Get Details if Needed

For tickets that need more detail, use `mcp__atlassian__jira_get_issue` to fetch full fields.

## Output Format

Present tickets grouped by status:

### In Progress
| Key | Summary | Priority | Updated |
|-----|---------|----------|---------|

### To Do
| Key | Summary | Priority | Created |
|-----|---------|----------|---------|

### Blocked / On Hold
| Key | Summary | Blocker Reason |
|-----|---------|----------------|

### Recently Completed (last 7 days)
| Key | Summary | Completed |
|-----|---------|-----------|

## Summary Stats
- Total active tickets: X
- Oldest ticket age: X days
- Tickets updated today: X

## Tips
- Highlight any tickets not updated in 7+ days
- Flag high-priority tickets that need attention
- Note any tickets missing story points

## Notes

- Apply the ghostwriter skill for tone
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
