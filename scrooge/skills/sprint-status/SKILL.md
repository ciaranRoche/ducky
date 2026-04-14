---
name: sprint-status
description: Sprint health overview for team leads - progress, blockers, and risks
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_batch_get_changelogs
argument-hint: [project-key]
---

# Sprint Status (Team Lead View)

Comprehensive sprint health report for team leads and scrum masters. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Arguments
- `$1` (optional): Project key. Defaults to `HYPERFLEET`.

## Story Points Field Mapping

The JIRA instance stores story points in custom fields. When reading issue data from MCP tools, look for these fields (use the first one that has a value):

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen / Jira Software field — check this first |
| `customfield_10028` | Story Points | Classic field |

If neither field has a value, treat the issue as having 0 story points and note it in the report.

## Behavior

### 1. Find the Active Sprint

- Use `mcp__atlassian__jira_get_agile_boards` to find the board for the project
- Use `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint (state = "active")
- If no active sprint, report that and stop

### 2. Gather Sprint Data

- Use `mcp__atlassian__jira_get_sprint_issues` to pull all issues in the active sprint
- For each issue, note: key, summary, status, assignee, story points, priority, issue type
- Group issues by status (To Do, In Progress, Done) from the returned data

### 3. Identify Risks

- **Blockers**: Use `mcp__atlassian__jira_search` with JQL: `project = HYPERFLEET AND priority in (Highest, High) AND status != Done AND sprint in openSprints()`
- **Unassigned**: Filter sprint issues where assignee is null
- **Stale in-progress**: Use `mcp__atlassian__jira_batch_get_changelogs` on in-progress issues to detect items without recent status changes
- **Carry-over risk**: Issues in To Do with high story points late in the sprint

### 4. Team Workload

- Group issues by assignee from the sprint issues data
- Show points assigned vs points completed per person
- Flag anyone with disproportionate load or 0 completed points

## Output Format

### Sprint Overview
```
Sprint: [Sprint Name]
Duration: [Start Date] - [End Date]
Days Remaining: X days
```

### Progress Summary
| Status | Count | Story Points |
|--------|-------|--------------|
| To Do | X | X pts |
| In Progress | X | X pts |
| Done | X | X pts |
| **Total** | **X** | **X pts** |

Progress: [=========>    ] 65% complete

### Risk Assessment

#### Blockers & High Priority
- TICKET-1: [Summary] - Assigned to [Name] - [X days in status]
- TICKET-2: [Summary] - **UNASSIGNED**

#### At-Risk Items
- Tickets in progress > 5 days without update
- Tickets without story points
- Unassigned tickets

#### Carry-Over Risk
- Tickets likely to spill: X
- Story points at risk: X

### Team Workload
| Team Member | To Do | In Progress | Done |
|-------------|-------|-------------|------|
| [Name] | X | X | X |

### Recommendations
- List actionable items for the team lead
- Highlight tickets needing attention
- Suggest re-assignments if workload is unbalanced

## Notes

- Apply the ghostwriter skill for tone
- Keep the report scannable — tables over prose
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
