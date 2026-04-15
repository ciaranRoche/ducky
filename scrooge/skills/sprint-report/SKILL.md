---
name: sprint-report
description: Daily sprint health report — progress, velocity, blockers, at-risk tickets,
  and team workload.
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_get_board_issues
  - mcp__atlassian__jira_batch_get_changelogs
  - mcp__atlassian__jira_get_issue_dates
---

# Sprint Report: Daily Sprint Health

Generate a sprint health report for the current active sprint. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Story Points Field Mapping

The JIRA instance stores story points in custom fields. When reading issue data from MCP tools, look for these fields (use the first one that has a value):

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen / Jira Software field — check this first |
| `customfield_10028` | Story Points | Classic field |

If neither field has a value, treat the issue as having 0 story points and note it in the report.

**Important:** These custom fields are NOT returned by default by MCP tools. When calling `mcp__atlassian__jira_get_sprint_issues`, `mcp__atlassian__jira_search`, or `mcp__atlassian__jira_get_issue`, you **must** include them in the `fields` parameter:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,customfield_10016,customfield_10028"
```

## Behavior

### 1. Find the Active Sprint

- Use `mcp__atlassian__jira_get_agile_boards` to find the board for project HYPERFLEET
- Use `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint (state = "active")
- If no active sprint, report that and stop

### 2. Gather Sprint Data

- Use `mcp__atlassian__jira_get_sprint_issues` to pull all issues in the active sprint
- For each issue, note: key, summary, status, assignee, story points, priority, issue type
- Use `mcp__atlassian__jira_batch_get_changelogs` on sprint issues to detect recent status transitions (last 24h)

### 3. Calculate Metrics

- **Progress**: count issues by status category (To Do, In Progress, Done) and sum story points per category
- **Velocity**: points completed so far vs total sprint capacity
- **Burn rate**: points completed per day vs required rate to finish on time
- **Sprint dates**: start date, end date, days remaining

### 4. Identify Risks

- **Blockers**: issues with status "Blocked" or flagged
- **Stale in-progress**: issues in "In Progress" for 3+ days without status change (use changelogs)
- **Unassigned**: issues in the sprint with no assignee
- **Large uncommitted**: issues with 8+ story points still in To Do
- **Scope creep**: issues added to the sprint after sprint start (use changelogs)

### 5. Team Workload

- Group issues by assignee
- Show points assigned vs points completed per person
- Flag anyone with 0 completed points or disproportionate load

## Output Format

```
## Sprint Report: [Sprint Name]
**Day [X] of [Y]** | [start] - [end] | [days remaining] days left

### Progress
| Status | Issues | Points |
|--------|--------|--------|
| Done   | X      | Y      |
| In Progress | X | Y     |
| To Do  | X      | Y      |
| **Total** | **X** | **Y** |

Burn rate: X pts/day (need Y pts/day to finish on time)

### Risks
- [PROJ-123] Blocked: [summary] — blocked for X days
- [PROJ-456] Stale: [summary] — in progress for X days, no movement
- [PROJ-789] Unassigned: [summary] — X points at risk

### Team Workload
| Assignee | Assigned | Done | Remaining |
|----------|----------|------|-----------|
| Name     | X pts    | Y pts| Z pts     |

### Key Movements (last 24h)
- [PROJ-123] moved to Done (X pts)
- [PROJ-456] moved to In Progress
```

## Notes

- Apply the ghostwriter skill for tone
- Keep the report scannable — tables over prose
- If a metric can't be calculated (e.g., no story points on issues), note it and move on
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
