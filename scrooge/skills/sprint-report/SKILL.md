---
name: sprint-report
description: Sprint health report — progress, burn rate, blockers, at-risk tickets,
  carry-over risk, and team workload.
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_get_board_issues
  - mcp__atlassian__jira_batch_get_changelogs
  - mcp__atlassian__jira_get_issue_dates
argument-hint: '[project-key]'
---

# Sprint Report

Comprehensive sprint health report. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Arguments
- `$1` (optional): Project key. Defaults to `HYPERFLEET`.

## Custom Field Reference

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen — check first |
| `customfield_10028` | Story Points | Classic fallback |

If neither field has a value, treat the issue as having 0 story points and note it in the report.

**Important:** Pass the `fields` parameter on every issue query:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,customfield_10016,customfield_10028"
```

## Behavior

### 1. Find the Active Sprint

- Use `mcp__atlassian__jira_get_agile_boards` to find the board for the project
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

- **Blockers**: issues with status "Blocked" or flagged, plus Highest/High priority items not Done
- **Stale in-progress**: issues in "In Progress" for 3+ days without status change (use changelogs)
- **Unassigned**: issues in the sprint with no assignee
- **Large uncommitted**: issues with 8+ story points still in To Do
- **Scope creep**: issues added to the sprint after sprint start (use changelogs)
- **Unpointed**: issues with no story points assigned

### 5. Carry-Over Risk

Identify items likely to spill into the next sprint:
- Issues still in To Do with no recent activity
- Large items (8+ pts) in To Do or early In Progress late in the sprint (past 60% of sprint duration)
- In Progress items with stale changelogs

Report: total items at risk of carry-over, total story points at risk.

### 6. Team Workload

- Group issues by assignee
- Show points assigned vs points completed per person
- Flag anyone with 0 completed points or disproportionate load

### 7. Key Movements (Last 24h)

From the changelog data, surface:
- Issues that moved to Done (points completed)
- Issues that moved to In Progress (work started)
- Issues that were added to or removed from the sprint

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

Progress: [=========>    ] X% (by points)
Burn rate: X pts/day (need Y pts/day to finish on time)

### Risks
- [PROJ-123] Blocked: [summary] — blocked for X days
- [PROJ-456] Stale: [summary] — in progress for X days, no movement
- [PROJ-789] Unassigned: [summary] — X points at risk
- Scope creep: X issues added after sprint start

### Carry-Over Risk
- Items likely to spill: X
- Story points at risk: X pts
| Ticket | Summary | Pts | Status | Why |
|--------|---------|-----|--------|-----|
| PROJ-123 | [Summary] | 8 | To Do | Large, no activity |
| PROJ-456 | [Summary] | 5 | In Progress | Stale 4 days |

### Team Workload
| Assignee | Assigned | Done | Remaining |
|----------|----------|------|-----------|
| Name     | X pts    | Y pts| Z pts     |

### Key Movements (last 24h)
- [PROJ-123] moved to Done (X pts)
- [PROJ-456] moved to In Progress
- [PROJ-789] added to sprint (scope change)

### Recommendations
- [Actionable item for the team — e.g., "Unblock PROJ-123 to recover 8 pts"]
- [Rebalance suggestion if workload is skewed]
- [Scope discussion if burn rate is behind]
```

## Notes

- Apply the ghostwriter skill for tone
- Keep the report scannable — tables over prose
- If a metric can't be calculated (e.g., no story points on issues), note it and move on
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
