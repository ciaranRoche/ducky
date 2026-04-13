---
name: backlog-hygiene
description: Weekly backlog audit — field completeness, staleness tiers, and duplicate
  detection.
allowed-tools:
  - mcp__atlassian__search
  - mcp__atlassian__get_issue
  - mcp__atlassian__get_project_components
  - mcp__atlassian__batch_get_changelogs
  - mcp__atlassian__get_issue_dates
---

# Backlog Hygiene: Weekly Backlog Audit

Audit the full backlog for field completeness, staleness, and potential duplicates. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Story Points Field Mapping

The JIRA instance stores story points in custom fields. When reading issue data from MCP tools, look for these fields (use the first one that has a value):

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen / Jira Software field — check this first |
| `customfield_10028` | Story Points | Classic field |

If neither field has a value, treat the issue as unpointed.

## Behavior

### 1. Pull the Backlog

Search using `mcp__atlassian__search` with JQL:
```
project = HYPERFLEET AND status != Done AND status != Closed AND (sprint not in openSprints() OR sprint is EMPTY) ORDER BY updated ASC
```

Paginate if needed (use `startAt` parameter).

### 2. Field Completeness Checks

For each backlog issue, check these 6 fields:

| Field | Check |
|-------|-------|
| **Description** | Exists and has meaningful content (> 20 chars) |
| **Priority** | Set to something other than the default |
| **Story points** | Assigned (for Stories and Bugs) |
| **Component** | Set and valid (verify against `mcp__atlassian__get_project_components`) |
| **Acceptance criteria** | Present in description for Stories |
| **Issue type** | Not "Task" when it should be a Story or Bug |

Score each issue: X/6 fields complete.

### 3. Staleness Tiers

Classify issues by time since last update:

| Tier | Criteria | Action |
|------|----------|--------|
| **Stale** | No updates in 90+ days | Consider closing or re-evaluating |
| **Aging** | No updates in 60-89 days | Needs attention next grooming |
| **Cooling** | No updates in 30-59 days | Monitor |
| **Active** | Updated within 30 days | No action needed |

Use `mcp__atlassian__batch_get_changelogs` to check for meaningful updates (status changes, comment additions), not just field edits.

### 4. Duplicate Detection

Group issues by similarity:
- Compare summaries for near-duplicates (same keywords, similar phrasing)
- Flag issues in the same component with overlapping descriptions
- Present potential duplicates as pairs with a confidence note

This is heuristic — flag likely duplicates, don't assert.

### 5. Backlog Health Score

Calculate overall metrics:
- Total open issues in backlog
- Average field completeness (X/6)
- Distribution across staleness tiers
- Number of potential duplicates
- Issues with 0 story points (for Stories/Bugs)

## Output Format

```
## Backlog Hygiene Report
**[X] open issues in backlog** | Avg completeness: [X]/6

### Health Summary
| Metric | Count | % |
|--------|-------|---|
| Fully groomed (6/6) | X | Y% |
| Needs work (3-5/6) | X | Y% |
| Incomplete (0-2/6) | X | Y% |

### Staleness
| Tier | Count | Oldest |
|------|-------|--------|
| Stale (90+ days) | X | [PROJ-123] (X days) |
| Aging (60-89 days) | X | [PROJ-456] (X days) |
| Cooling (30-59 days) | X | |
| Active (< 30 days) | X | |

### Top Issues to Fix
1. [PROJ-123] — stale (120 days), missing description and points
2. [PROJ-456] — missing acceptance criteria, no component
3. [PROJ-789] — possible duplicate of [PROJ-012]
...

### Potential Duplicates
| Issue A | Issue B | Why |
|---------|---------|-----|
| [PROJ-123] | [PROJ-456] | Both reference "adapter timeout handling" |

### Incomplete Issues (0-2/6 fields)
| Ticket | Type | Summary | Missing |
|--------|------|---------|---------|
| [PROJ-123] | Story | ... | description, points, AC |
```

## Notes

- Apply the ghostwriter skill for tone
- Cap "Top Issues to Fix" at 10 — prioritize by staleness + incompleteness
- Duplicate detection is best-effort. Use cautious language ("possible duplicate", "similar to")
- This runs weekly (Sunday evening) to prep for Monday grooming
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
