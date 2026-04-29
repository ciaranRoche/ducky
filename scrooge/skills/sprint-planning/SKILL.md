---
name: sprint-planning
description: Sprint planning prep — backlog candidates, stale items, release priorities,
  capacity analysis, and pull suggestions. Run before sprint planning to prepare.
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_versions
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_batch_get_changelogs
---

# Sprint Planning Prep

Surface everything needed to run an effective sprint planning session: backlog candidates ready to pull, stale items at risk of being lost, release-scoped priorities, and capacity analysis. Read-only — surfaces data, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

Run this before sprint planning to anchor the conversation around what to commit to next.

## Custom Field Reference

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen — check first |
| `customfield_10028` | Story Points | Classic fallback |
| `customfield_10464` | Activity Type | Select dropdown |

**Important:** Pass the `fields` parameter on every issue query:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,fixVersions,customfield_10016,customfield_10028,customfield_10464"
```

## Behavior

### 1. Current Sprint Context

Establish where things stand before planning the next sprint.

- Use `mcp__atlassian__jira_get_agile_boards` → `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint
- Use `mcp__atlassian__jira_get_sprint_issues` to pull all sprint issues. **You must pass the `fields` parameter:**
  ```
  fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,fixVersions,customfield_10016,customfield_10028,customfield_10464"
  ```
- For each issue, read story points from `customfield_10016` first, then fall back to `customfield_10028`.
- Calculate: issues by status, story points per status, progress %
- Identify carry-over candidates: items still in To Do or In Progress that may spill

This gives the baseline: how much is likely to carry over and how much capacity is actually available.

### 2. Historical Velocity

Look at the most recent closed sprint(s) to establish a velocity baseline.

- Use `mcp__atlassian__jira_get_sprints_from_board` — find the last 1-2 closed sprints (state = "closed")
- For each, use `mcp__atlassian__jira_get_sprint_issues` (pass `fields` parameter with `customfield_10016,customfield_10028`) to count completed story points
- Calculate average velocity (points completed per sprint)

This sets the capacity ceiling for the next sprint.

### 3. Release-Scoped Priorities

Surface items tagged with a Fix Version that need sprint assignment.

- Use `mcp__atlassian__jira_get_project_versions` with `project_key: HYPERFLEET` to find unreleased versions
- For each unreleased version with a release date, search:
  ```
  project = HYPERFLEET AND fixVersion = "VERSION_NAME" AND status != Done AND status != Closed AND (sprint not in openSprints() OR sprint is EMPTY) ORDER BY priority ASC
  ```
- Rank by release date urgency — items for the nearest release come first
- Note readiness: does the item have story points, assignee, AC?

### 4. Backlog Candidates

Surface groomed backlog items that are ready to pull into a sprint.

Search using `mcp__atlassian__jira_search` with the `fields` parameter:
```
jql: "project = HYPERFLEET AND status != Done AND status != Closed AND (sprint not in openSprints() OR sprint is EMPTY) ORDER BY priority ASC, updated DESC"
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,fixVersions,customfield_10016,customfield_10028,customfield_10464"
```

For each item, score readiness on 6 points (story points, description, AC, component, activity type, assignee). Filter to items scoring 4+ — these are ready or nearly ready.

Cap at 15 candidates. Sort by priority, then by readiness score.

### 5. Stale Items at Risk

Surface backlog items that are aging and at risk of being forgotten.

From the backlog query results, use `mcp__atlassian__jira_batch_get_changelogs` on items with `updated` older than 60 days to verify staleness is real (meaningful updates, not just bot edits).

Classify:
| Tier | Criteria | Recommended Action |
|------|----------|-------------------|
| **Stale** | No meaningful update in 90+ days | Close or re-evaluate — is this still needed? |
| **Aging** | No meaningful update in 60-89 days | Groom next session or deprioritize |

Cap at 10 items. These need a decision: pull them in, close them, or consciously defer them.

### 6. Ungroomed Items Needing Attention

Surface items that can't be pulled in yet because they're missing critical fields.

From the backlog query, filter to items scoring 0-3 on the readiness scale that have High or Highest priority. These are important but not sprint-ready.

For each, note what's missing (no points, no AC, no assignee, etc.).

Cap at 10 items.

### 7. Capacity Analysis

Estimate available capacity for the next sprint:

- **Velocity baseline**: average points completed per sprint (from Step 2)
- **Carry-over load**: points likely carrying over from current sprint (from Step 1)
- **Available capacity**: velocity minus carry-over
- **Release pressure**: total remaining points across unreleased Fix Versions with dates (from Step 3)

This is an estimate, not a commitment. PTO and unexpected work aren't factored in — note this caveat.

## Output Format

```
## Sprint Planning Prep
**Generated:** [date]
**Current sprint:** [Sprint Name] (Day X of Y, X% complete)

---

### 1. Capacity Estimate

| Metric | Points |
|--------|--------|
| Average velocity (last 2 sprints) | X pts |
| Likely carry-over | X pts |
| **Available capacity** | **X pts** |

**Release pressure:** X pts remaining across [N] unreleased versions

---

### 2. Carry-Over from Current Sprint

| Ticket | Summary | Pts | Status | Assignee |
|--------|---------|-----|--------|----------|
| PROJ-123 | [Summary] | 5 | In Progress | @person |
| PROJ-456 | [Summary] | 3 | To Do | @person |

**Total carry-over:** X items, Y pts

---

### 3. Release Priorities (not in sprint)

Items tagged with a Fix Version that need sprint assignment:

#### [Version Name] — release date: [date] ([X] days away)
| Ticket | Summary | Pts | Priority | Ready? |
|--------|---------|-----|----------|--------|
| PROJ-789 | [Summary] | 5 | High | 5/6 |
| PROJ-012 | [Summary] | - | High | 2/6 (no pts, no AC) |

---

### 4. Backlog Candidates (ready to pull)

Groomed items not in any sprint, scored 4+/6:

| Ticket | Summary | Pts | Priority | Score | Missing |
|--------|---------|-----|----------|-------|---------|
| PROJ-345 | [Summary] | 3 | High | 6/6 | — |
| PROJ-678 | [Summary] | 5 | Medium | 4/6 | labels, assignee |

---

### 5. Stale Items (at risk of being lost)

| Ticket | Summary | Last Meaningful Update | Days Stale | Action Needed |
|--------|---------|----------------------|------------|---------------|
| PROJ-111 | [Summary] | 2025-12-01 | 143 | Close or re-evaluate |
| PROJ-222 | [Summary] | 2026-01-15 | 98 | Groom or deprioritize |

---

### 6. Ungroomed but Important

High-priority items that can't be pulled in yet:

| Ticket | Summary | Priority | Score | Missing |
|--------|---------|----------|-------|---------|
| PROJ-333 | [Summary] | Highest | 2/6 | pts, AC, component, activity type |

---

### Suggested Pull Order

Based on release pressure, priority, and readiness:

| # | Ticket | Summary | Pts | Why |
|---|--------|---------|-----|-----|
| 1 | PROJ-123 | [Summary] | 5 | Carry-over, in progress |
| 2 | PROJ-789 | [Summary] | 5 | Release [v1.2] in 14 days |
| 3 | PROJ-345 | [Summary] | 3 | High priority, fully groomed |
| 4 | PROJ-678 | [Summary] | 5 | Medium priority, nearly ready |

**Running total:** X pts (vs X pts available)
```

## Integration

- **sprint-review**: Run after planning to validate the final sprint — readiness, capacity, risks
- **hygiene**: Use to audit sprint tickets after planning is complete
- **ticket-triage**: Use to walk through ungroomed items interactively
- **story-pointer**: Use to estimate unpointed candidates
- **set-activity-type**: Use to fill missing activity types
- **release-status**: Complementary — release-status gives a deep dive on one Fix Version, this gives a cross-release view for planning
- **backlog-grooming**: Complementary — backlog-grooming is the full weekly audit, this surfaces planning-relevant items

## Notes

- Apply the ghostwriter skill for tone
- The capacity estimate is directional — note that PTO, support load, and unplanned work aren't factored in
- The suggested pull order is a starting point for discussion, not a prescribed commitment
- Keep the report scannable — tables over prose
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
