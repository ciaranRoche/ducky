---
name: release-status
description: Release readiness report — progress, risks, remaining work, and priorities
  by Fix Version. Activates when users ask about release status, what's left for a
  release, or release readiness.
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_versions
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_batch_get_changelogs
argument-hint: '[fix-version]'
---

# Release Status

Assess the readiness of a release by Fix Version. Shows what's done, what's remaining, risks, and what to prioritize. Read-only — surfaces data, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Arguments
- `$1` (optional): Fix Version name (e.g., `v1.2.0`). If omitted, auto-detect the next unreleased version.

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

### 1. Determine the Target Version

**If version argument provided:**
- Use `mcp__atlassian__jira_get_project_versions` with `project_key: HYPERFLEET`
- Find the version matching the argument name
- If not found, report the error and list available versions

**If no argument:**
- Use `mcp__atlassian__jira_get_project_versions` with `project_key: HYPERFLEET`
- Filter to unreleased versions (`released = false`)
- Pick the one with the earliest release date
- If no release dates are set, pick the first unreleased version
- If no unreleased versions exist, report that and suggest creating one

Note the version name, description, release date (if set), and status.

### 2. Pull All Issues for the Version

Search using `mcp__atlassian__jira_search` with JQL:
```
project = HYPERFLEET AND fixVersion = "VERSION_NAME" ORDER BY priority ASC, status ASC
```

Pass the `fields` parameter. Paginate if needed.

### 3. Determine Active Sprint Context

- Use `mcp__atlassian__jira_get_agile_boards` to find the board for project HYPERFLEET
- Use `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint
- Use `mcp__atlassian__jira_get_sprint_issues` to get active sprint issue keys (pass `fields` parameter)
- Cross-reference: for each release issue, note whether it is currently in the active sprint

### 4. Classify Issues

Group all release issues by status:
- **Done**: Completed work
- **In Progress**: Actively being worked on
- **To Do**: Not yet started (includes any non-Done, non-In Progress status)

For each issue, record: key, summary, type, points, priority, status, assignee, whether it's in the active sprint.

### 5. Score Remaining Items for Readiness

For each non-Done issue, evaluate readiness on a 6-point scale:

| Criterion | Check |
|-----------|-------|
| **Story Points** | `customfield_10016` or `customfield_10028` has a value |
| **Description** | Description exists and has > 100 characters |
| **Acceptance Criteria** | Description contains "acceptance criteria", "AC:", "given/when/then", or checklist pattern (- [ ]) |
| **Component** | Components array is non-empty and valid (validate against `mcp__atlassian__jira_get_project_components`) |
| **Activity Type** | `customfield_10464` is set |
| **Assignee** | Assignee is set |

Tier results:
- **Ready** (5-6): Can be picked up immediately
- **Nearly Ready** (3-4): Minor gaps to address
- **Needs Grooming** (0-2): Significant work before sprint-ready

### 6. Identify Risks

Check remaining items for:
- **Unpointed**: No story points — can't estimate release effort
- **Unassigned**: No assignee — no owner means no progress
- **Blocked/Stale**: Use `mcp__atlassian__jira_batch_get_changelogs` on remaining items to detect issues with no status change in 7+ days
- **Large items**: 8+ story points — risk of spilling across sprints
- **Not in sprint**: Remaining items not in the active sprint — need to be planned

### 7. Find Potentially Untagged Items

Query for items that might belong to this release but aren't tagged:
```
project = HYPERFLEET AND fixVersion is EMPTY AND status != Done AND status != Closed ORDER BY priority ASC, updated DESC
```

Filter to items sharing components or labels with the release items. Cap at 10. Present as "Might these belong to [version]?" — speculative, not assertive.

If no release items have components or labels to match against, skip this section.

## Output Format

```
## Release Status: [Version Name]
**Release date:** [date or "Not set"]
**Description:** [version description or "None"]
**Generated:** [date]

---

### Release Overview

| Metric | Count | Points |
|--------|-------|--------|
| Total scope | X | Y pts |
| Done | X | Y pts |
| In Progress | X | Y pts |
| To Do | X | Y pts |
| **Remaining** | **X** | **Y pts** |

Progress: [====>-----] X% complete (by points)

---

### What's Remaining

| Ticket | Summary | Type | Pts | Priority | Status | Assignee | In Sprint? |
|--------|---------|------|-----|----------|--------|----------|------------|
| PROJ-123 | [Summary] | Story | 5 | High | To Do | @person | Yes |
| PROJ-456 | [Summary] | Bug | 3 | High | In Progress | @person | Yes |
| PROJ-789 | [Summary] | Story | - | Medium | To Do | - | No |

**Total remaining:** X items, Y points (Z unpointed)

---

### Readiness Assessment

| Tier | Count | Points |
|------|-------|--------|
| Ready (5-6/6) | X | Y pts |
| Nearly Ready (3-4/6) | X | Y pts |
| Needs Grooming (0-2/6) | X | Y pts |

#### Nearly Ready (with gaps)
| Ticket | Summary | Score | Missing |
|--------|---------|-------|---------|
| PROJ-789 | [Summary] | 4/6 | story points, assignee |

---

### Risks

| Risk | Count | Items |
|------|-------|-------|
| Unpointed | X | PROJ-789, PROJ-012 |
| Unassigned | X | PROJ-789 |
| Stale (7+ days no update) | X | PROJ-345 |
| Large (8+ pts) | X | PROJ-456 |
| Not in active sprint | X | PROJ-789, PROJ-012 |

[If release date is set and approaching:]
**Release date is [date] ([X] days away) with [Y] points remaining.**

---

### Might These Belong to [Version]?

| Ticket | Summary | Type | Pts | Shared |
|--------|---------|------|-----|--------|
| PROJ-999 | [Summary] | Bug | 3 | component: platform |

---

### Next Steps
- [ ] Groom "Nearly Ready" items (use `/scrooge:ticket-triage`)
- [ ] Point unpointed items (use `/scrooge:story-pointer`)
- [ ] Set missing activity types (use `/scrooge:set-activity-type`)
- [ ] Review untagged items — add to release if they belong
- [ ] Pull top priorities into the next sprint
```

## Integration

- **ticket-triage**: Use to walk through "Nearly Ready" release items interactively
- **story-pointer**: Use to estimate unpointed release items
- **set-activity-type**: Use to fill missing activity types on release items
- **sprint-planning**: Complementary — sprint-planning gives a cross-release view for planning, this gives a deep dive on one Fix Version
- **sprint-report**: Complementary — sprint-report tracks the active sprint, this tracks the release
- **backlog-grooming**: Complementary — backlog-grooming audits the full backlog, this focuses on one release version

## Notes

- Apply the ghostwriter skill for tone
- Keep the report scannable — tables over prose
- This skill surfaces candidates and priorities — it does NOT recommend a sprint commitment total
- If the release has many Done items, keep that section compact (count + points, not a full list)
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
