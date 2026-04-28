---
name: sprint-review
description: Pre-sprint readiness check — validate committed items, flag risks, check
  capacity, and surface anything that might have been missed. Run after sprint planning,
  before the sprint starts.
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_get_project_versions
  - mcp__atlassian__jira_batch_get_changelogs
---

# Sprint Review: Pre-Sprint Readiness Check

Validate the sprint after planning is complete. Catches items that aren't ready to be worked, flags risks to watch for during execution, and surfaces important items that may have been missed. Read-only — surfaces data, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

Run this after sprint planning, before the sprint kicks off.

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

### 1. Get Sprint Contents

- Use `mcp__atlassian__jira_get_agile_boards` → `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint (or next sprint if one exists)
- Use `mcp__atlassian__jira_get_sprint_issues` to pull all committed issues (pass `fields` parameter)
- Note sprint name, start date, end date, and goal (if set)

### 2. Readiness Audit

For each sprint item, score readiness on a 6-point scale:

| # | Check | Requirement |
|---|-------|-------------|
| 1 | Story Points | `customfield_10016` or `customfield_10028` has a value |
| 2 | Description | Exists and has > 100 characters |
| 3 | Acceptance Criteria | Contains testable criteria (AC:, given/when/then, checklist) |
| 4 | Component | Set and valid (validate against `mcp__atlassian__jira_get_project_components`) |
| 5 | Activity Type | `customfield_10464` is set |
| 6 | Assignee | Assigned to someone |

Classify each item:
- **Ready** (5-6/6): Can be picked up immediately
- **Gaps** (3-4/6): Needs minor fixes before work starts
- **Not Ready** (0-2/6): Should not be in the sprint without grooming

Flag any item scoring below 5 — these need attention before the sprint starts.

### 3. Watch List

Surface risks to monitor during sprint execution:

- **Large items** (8+ pts): Risk of spilling. Is there a break-down plan?
- **Unassigned items**: No owner means no progress on day 1
- **External dependencies**: Items mentioning other teams, external APIs, or blocked-by links
- **Stale items**: Items pulled from backlog that haven't been updated in 30+ days — context may be stale. Use `mcp__atlassian__jira_batch_get_changelogs` to check for meaningful recent activity.
- **13-point items**: These should have been split. Flag explicitly.
- **Items without Fix Version**: Not tied to a release — may lack urgency or priority context
- **Carry-over items**: Items that were in the previous sprint and carried over. Check if they've been re-scoped or if the same blockers persist.

### 4. Capacity Check

Estimate whether the sprint is over- or under-committed:

- **Total committed**: Sum all story points in the sprint
- **Historical velocity**: Check the last 2 closed sprints via `mcp__atlassian__jira_get_sprints_from_board` (state = "closed") and `mcp__atlassian__jira_get_sprint_issues` for each. Calculate average points completed.
- **Comparison**: committed vs velocity. Flag if committed exceeds velocity by more than 20% (overcommitted) or is below 70% (undercommitted).

### 5. Activity Type Balance

Check the distribution of work across activity type categories:

| Category | Activity Types |
|----------|---------------|
| Reactive | Associate Wellness & Development, Incidents & Support, Security & Compliance |
| Core | Quality / Stability / Reliability |
| Proactive | Future Sustainability, Product / Portfolio Work |

Surface the point distribution. Flag if:
- 100% of points are in one category (no balance)
- Zero proactive work (the team isn't investing in the future)
- Reactive work is missing when there are known open incidents/CVEs
- Any items have no Activity Type set (uncategorized)

This is informational — the right balance depends on the team's situation.

### 6. Missing Items

Surface important items that are NOT in the sprint but probably should be:

**Release-scoped items not in sprint:**
- Use `mcp__atlassian__jira_get_project_versions` to find unreleased versions with release dates
- Search for items with those Fix Versions that aren't in the sprint and aren't Done:
  ```
  project = HYPERFLEET AND fixVersion = "VERSION" AND status != Done AND status != Closed AND sprint not in openSprints() ORDER BY priority ASC
  ```
- Flag items for releases within 30 days that haven't been planned

**High-priority backlog items:**
- Search for Highest/High priority items not in any sprint:
  ```
  project = HYPERFLEET AND priority in (Highest, High) AND status != Done AND status != Closed AND (sprint not in openSprints() OR sprint is EMPTY) ORDER BY priority ASC, created ASC
  ```
- Cap at 5 items. These may have been intentionally deferred, but worth a conscious check.

## Output Format

```
## Sprint Review: [Sprint Name]
**Sprint:** [start] — [end] ([X] days)
**Goal:** [sprint goal or "Not set"]
**Generated:** [date]

---

### 1. Sprint Summary

| Metric | Value |
|--------|-------|
| Total items | X |
| Total points | X pts |
| Ready (5-6/6) | X items (Y pts) |
| Gaps (3-4/6) | X items (Y pts) |
| Not Ready (0-2/6) | X items (Y pts) |

---

### 2. Items Needing Attention

Items scoring below 5/6 that need fixes before the sprint starts:

| Ticket | Summary | Score | Missing |
|--------|---------|-------|---------|
| PROJ-123 | [Summary] | 3/6 | AC, component, activity type |
| PROJ-456 | [Summary] | 4/6 | assignee |

---

### 3. Watch List

Risks to monitor during sprint execution:

| Risk | Ticket | Summary | Detail |
|------|--------|---------|--------|
| Large (8+ pts) | PROJ-789 | [Summary] | 13 pts — should this be split? |
| Unassigned | PROJ-456 | [Summary] | No owner |
| Stale | PROJ-012 | [Summary] | Last meaningful update 45 days ago |
| Carry-over | PROJ-345 | [Summary] | 2nd sprint — same blockers? |
| No Fix Version | PROJ-678 | [Summary] | Not tied to a release |

---

### 4. Capacity

| Metric | Points |
|--------|--------|
| Committed | X pts |
| Avg velocity (last 2 sprints) | X pts |
| Delta | +/- X pts |

**[OVERCOMMITTED / ON TARGET / UNDERCOMMITTED]**
[If overcommitted: "Committed X% above velocity. Consider descoping Y pts to reduce carry-over risk."]
[If undercommitted: "Room for X more points. See Missing Items below."]

---

### 5. Activity Type Balance

| Category | Points | % |
|----------|--------|---|
| Reactive | X | Y% |
| Core (Quality/Stability) | X | Y% |
| Proactive | X | Y% |
| Uncategorized | X | Y% |

[Flags if any — e.g., "No proactive work planned", "3 items missing Activity Type"]

---

### 6. Missing Items

#### Release-scoped (not in sprint)
| Ticket | Summary | Fix Version | Release Date | Priority |
|--------|---------|-------------|-------------|----------|
| PROJ-111 | [Summary] | v1.2.0 | [date] | High |

#### High-priority backlog
| Ticket | Summary | Priority | Days in Backlog |
|--------|---------|----------|-----------------|
| PROJ-222 | [Summary] | Highest | 14 |

---

### Verdict

- **GO** — Sprint is well-planned. Items are groomed, capacity is reasonable, risks are manageable.
- **GO WITH CAVEATS** — Sprint can start, but address [X items needing attention] in the first day. Watch [specific risks].
- **HOLD** — [X] items are not ready. Capacity is [over/under] by [Y]%. Recommend addressing gaps before starting.
```

## Integration

- **sprint-planning**: Run before this skill — sprint-planning prepares candidates, sprint-review validates the final plan
- **hygiene**: Complementary — sprint-review runs readiness checks inline, hygiene gives a deeper single-ticket view
- **ticket-triage**: Use to fix items flagged as "Not Ready" or "Gaps"
- **story-pointer**: Use to estimate any unpointed items found
- **set-activity-type**: Use to fill uncategorized items
- **sprint-report**: Use during the sprint for ongoing health tracking

## Notes

- Apply the ghostwriter skill for tone
- Keep the report scannable — tables over prose
- The verdict is a recommendation, not a gate. The team decides whether to start.
- Carry-over detection: check if any sprint items were also in the previous closed sprint
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
