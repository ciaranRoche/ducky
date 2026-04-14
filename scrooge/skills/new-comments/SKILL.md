---
name: new-comments
description: Find tickets with new comments you may have missed
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
---

# New Comments

Find JIRA tickets with recent comments that the user should be aware of. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Behavior

### 1. Find Tickets with Recent Activity

Use `mcp__atlassian__jira_search` with JQL:
```
project = HYPERFLEET AND (assignee = currentUser() OR reporter = currentUser() OR watcher = currentUser()) AND updated >= -1d ORDER BY updated DESC
```

### 2. Check Comments on Each Ticket

For each ticket returned, use `mcp__atlassian__jira_get_issue` to retrieve full details including comments.

Filter to tickets that have comments added in the relevant time window.

## Output Format

### Tickets with Recent Activity

For each ticket with new comments:

**TICKET-KEY: Summary**
- Last updated: [timestamp]
- Latest comment by: [author]
- Comment preview: [first 100 chars of comment]

---

### Summary
- Tickets with new comments: X
- Comments requiring your response: X (where you were @mentioned)

## Notes
- Focus on tickets updated in the last 24 hours by default
- If user specifies a different timeframe (e.g., "last week"), adjust the JQL accordingly
- Highlight any comments that directly mention the user
- Flag urgent/blocking discussions
- Apply the ghostwriter skill for tone
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
