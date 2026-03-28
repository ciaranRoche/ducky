---
name: jira-my-tasks
description: List all your assigned JIRA tasks in the current project
allowed-tools: Bash
---

# My Tasks

Show all JIRA tickets currently assigned to the user in the configured project.

## Instructions

1. **Get all assigned tickets (recent, sorted by updated):**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND assignee = currentUser()" --order-by updated --reverse --plain 2>/dev/null
   ```

2. **For more detail with JSON output:**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND assignee = currentUser()" --order-by updated --reverse --raw 2>/dev/null | head -100
   ```

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

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`
