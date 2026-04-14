---
name: my-sprint
description: Show current sprint and your assigned tasks
allowed-tools:
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_search
---

# My Sprint

Show the current sprint information and the user's assigned tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Story Points Field Mapping

The JIRA instance stores story points in custom fields. When reading issue data from MCP tools, look for these fields (use the first one that has a value):

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen / Jira Software field — check this first |
| `customfield_10028` | Story Points | Classic field |

If neither field has a value, treat the issue as having 0 story points.

## Behavior

### 1. Find the Active Sprint

- Use `mcp__atlassian__jira_get_agile_boards` to find the board for project HYPERFLEET
- Use `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint (state = "active")
- If no active sprint, report that and stop

### 2. Get Sprint Issues

- Use `mcp__atlassian__jira_get_sprint_issues` to pull all issues in the active sprint
- Note sprint name, start date, end date, and goal (if available)

### 3. Get Your Assigned Tickets

- Use `mcp__atlassian__jira_search` with JQL:
  ```
  project = HYPERFLEET AND assignee = currentUser() AND sprint in openSprints() ORDER BY status ASC
  ```
- For each issue, note: key, summary, status, story points, priority, issue type

## Output Format

### Sprint Overview
- Sprint name and goal (if available)
- Days remaining in sprint
- Sprint progress indicator

### Your Tickets
Group by status:
- **To Do**: List tickets not yet started
- **In Progress**: List tickets being worked on
- **In Review/QA**: List tickets awaiting review
- **Done**: List completed tickets

### Summary
- Total tickets assigned: X
- Story points assigned: X
- Any blockers or high-priority items to highlight

## Notes

- Apply the ghostwriter skill for tone
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
