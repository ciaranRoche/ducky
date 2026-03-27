---
name: new-comments
description: Find tickets with new comments you may have missed
allowed-tools: Bash
---

# New Comments

Find JIRA tickets with recent comments that the user should be aware of.

## Instructions

1. **Find tickets you're involved with that have recent comments:**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND (assignee = currentUser() OR reporter = currentUser() OR watcher = currentUser()) AND updated >= -1d" --order-by updated --reverse --plain 2>/dev/null
   ```

2. **View specific ticket to see comments (for each relevant ticket):**
   ```bash
   jira issue view TICKET-KEY --comments 5 --plain 2>/dev/null
   ```

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

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`
