---
name: daily-report
description: Daily briefing — sprint progress, new tickets, and recent comments in
  one report. Run this to get oriented at the start of the day.
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
---

# Daily Report

One-shot daily briefing combining sprint progress, new ticket quality, and recent comments. Read-only — surfaces data, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Custom Field Reference

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen — check first |
| `customfield_10028` | Story Points | Classic fallback |
| `customfield_10464` | Activity Type | Select dropdown |

**Important:** Pass the `fields` parameter on every issue query:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,fixVersions,sprint,customfield_10016,customfield_10028,customfield_10464"
```

## Behavior

### 1. Sprint Progress Snapshot

- Use `mcp__atlassian__jira_get_agile_boards` → `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint
- Use `mcp__atlassian__jira_get_sprint_issues` to pull all sprint issues (pass `fields` parameter)
- Calculate: issues by status (To Do / In Progress / Done), story points per status, progress %
- Flag: blockers (Highest/High priority not Done), unassigned tickets, stale in-progress (3+ days no update based on `updated` field)

If no active sprint, note it and move to the next section.

### 2. New Tickets (Last 24h)

Search using `mcp__atlassian__jira_search` with JQL:
```
project = HYPERFLEET AND created >= -24h ORDER BY created DESC
```

If no tickets found, note it and move on.

For each new ticket, use `mcp__atlassian__jira_get_issue` to read full details (pass `fields` parameter). Assess grooming quality:

**By type:**
- **Bugs:** Steps to reproduce present, expected vs actual behavior, environment noted
- **Stories:** User story format or problem statement, acceptance criteria, reasonable scope
- **Tasks:** Clear definition of done, not a duplicate
- **Epics:** Success criteria defined, references child stories

**General quality:**
- Story points assigned
- Component set (validate against `mcp__atlassian__jira_get_project_components`)
- Labels present, sprint assigned, Fix Version set, Activity Type set

Classify each: **Ready** / **Needs Work** (1-2 gaps) / **Incomplete** (critical content missing)

### 3. New Comments

Search using `mcp__atlassian__jira_search` with JQL:
```
project = HYPERFLEET AND (assignee = currentUser() OR reporter = currentUser() OR watcher = currentUser()) AND updated >= -1d ORDER BY updated DESC
```

For each ticket, use `mcp__atlassian__jira_get_issue` to check for comments added in the last 24 hours. Filter to tickets that actually have new comments (not just field updates).

Highlight:
- Comments where you were @mentioned
- Urgent or blocking discussions
- Questions awaiting your response

If no new comments, note it and move on.

## Output Format

```
## Daily Report
**[Date]** | Sprint: [Sprint Name] (Day X of Y)

---

### Sprint Progress
| Status | Issues | Points |
|--------|--------|--------|
| Done | X | Y pts |
| In Progress | X | Y pts |
| To Do | X | Y pts |
| **Total** | **X** | **Y pts** |

Progress: [====>-----] X% (by points)

**Flags:** [blockers, unassigned, stale items — or "None"]

---

### New Tickets (last 24h)
**[X] tickets created**

| Ticket | Type | Summary | Status | Issues |
|--------|------|---------|--------|--------|
| PROJ-123 | Bug | Login fails | Incomplete | No steps to reproduce |
| PROJ-456 | Story | Add feature | Ready | — |

Summary: X ready, Y need work, Z incomplete

---

### New Comments
**[X] tickets with new activity**

**PROJ-123: [Summary]**
- Comment by [Author]: [first 100 chars]

**PROJ-456: [Summary]**
- @mentioned you: [first 100 chars]

---

### Action Items
- [ ] [Most important thing to address]
- [ ] [Second priority]
```

## Notes

- Apply the ghostwriter skill for tone
- Keep each section compact — if there's nothing to report in a section, one line is enough
- The sprint snapshot is intentionally brief — use `/scrooge:sprint-report` for the full health report
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
