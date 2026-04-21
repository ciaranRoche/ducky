---
name: release-prep
description: Release readiness assessment — progress, risks, and sprint planning priorities
  by Fix Version. Activates when users ask about release status, what's left for a
  release, or what to prioritize for sprint planning.
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

# Release Prep: Sprint Planning by Release

Assess the next release and surface what needs attention for sprint planning. Answers: "What's left for this release, how ready is it, and what should we prioritize?" Read-only — surfaces data, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

Run this before sprint planning to anchor the conversation around release deliverables.

## Arguments
- `$1` (optional): Fix Version name (e.g., `v1.2.0`). If omitted, auto-detect the next unreleased version.

## Story Points Field Mapping

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen / Jira Software field — check this first |
| `customfield_10028` | Story Points | Classic field |

## Activity Type Field

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10464` | Activity Type | Select dropdown |

**Important:** These custom fields are NOT returned by default by MCP tools. When calling `mcp__atlassian__jira_search` or `mcp__atlassian__jira_get_issue`, you **must** include them in the `fields` parameter:
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

Paginate if needed (use `startAt` parameter). Pull all issues — this is the full release scope.

### 3. Determine Active Sprint Context

- Use `mcp__atlassian__jira_get_agile_boards` to find the board for project HYPERFLEET
- Use `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint (state = "active")
- Use `mcp__atlassian__jira_get_sprint_issues` to get active sprint issue keys
- Cross-reference: for each release issue, note whether it is currently in the active sprint

### 4. Classify Issues

Group all release issues by status:
- **Done**: Completed work
- **In Progress**: Actively being worked on
- **To Do**: Not yet started (includes any non-Done, non-In Progress status)

For each issue, record: key, summary, type, points (customfield_10016 or customfield_10028), priority, status, assignee, whether it's in the active sprint.

### 5. Score Remaining Items for Readiness

For each non-Done issue, evaluate readiness on a 6-point scale:

| Criterion | Check |
|-----------|-------|
| **Story Points** | `customfield_10016` or `customfield_10028` has a value |
| **Description** | Description exists and has > 100 characters |
| **Acceptance Criteria** | Description contains "acceptance criteria", "AC:", "given/when/then", or checklist pattern (- [ ]) |
| **Component** | Components array is non-empty and contains valid components (validate against `mcp__atlassian__jira_get_project_components`) |
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

### 7. Determine Sprint Planning Priorities

Rank remaining items for sprint planning consideration (this is a suggested priority order, not a recommendation of what to commit):

1. **In Progress items** — finish what's started first
2. **Blocked items** — unblock to enable progress
3. **High-priority items not in sprint** — Highest/High priority remaining items in the backlog
4. **Release date pressure** — if the release date is approaching and significant work remains, flag it
5. **Remaining items by priority** — everything else, sorted by priority

### 8. Find Potentially Untagged Items

Query for items that might belong to this release but aren't tagged:
```
project = HYPERFLEET AND fixVersion is EMPTY AND status != Done AND status != Closed ORDER BY priority ASC, updated DESC
```

From these results, filter to items that share components or labels with the release items. Cap at 10 items. Present as "Might these belong to [version]?" — this is speculative, not assertive.

If no release items have components or labels to match against, skip this section.

## Output Format

```
## Release Prep: [Version Name]
**Release date:** [date or "Not set"]
**Description:** [version description or "None"]
**Generated:** [date]

---

### 1. Release Overview

| Metric | Count | Points |
|--------|-------|--------|
| Total scope | X | Y pts |
| Done | X | Y pts |
| In Progress | X | Y pts |
| To Do | X | Y pts |
| **Remaining** | **X** | **Y pts** |

Progress: [====>-----] X% complete (by points)

---

### 2. What's Remaining

| Ticket | Summary | Type | Pts | Priority | Status | Assignee | In Sprint? |
|--------|---------|------|-----|----------|--------|----------|------------|
| HYPERFLEET-123 | [Summary] | Story | 5 | High | To Do | @person | Yes |
| HYPERFLEET-456 | [Summary] | Bug | 3 | High | In Progress | @person | Yes |
| HYPERFLEET-789 | [Summary] | Story | - | Medium | To Do | - | No |

**Total remaining:** X items, Y points (Z unpointed)

---

### 3. Readiness Assessment

| Tier | Count | Points |
|------|-------|--------|
| Ready (5-6/6) | X | Y pts |
| Nearly Ready (3-4/6) | X | Y pts |
| Needs Grooming (0-2/6) | X | Y pts |

#### Nearly Ready (with gaps)
| Ticket | Summary | Score | Missing |
|--------|---------|-------|---------|
| HYPERFLEET-789 | [Summary] | 4/6 | story points, assignee |

---

### 4. Risks

| Risk | Count | Items |
|------|-------|-------|
| Unpointed | X | HYPERFLEET-789, HYPERFLEET-012 |
| Unassigned | X | HYPERFLEET-789 |
| Stale (7+ days no update) | X | HYPERFLEET-345 |
| Large (8+ pts) | X | HYPERFLEET-456 |
| Not in active sprint | X | HYPERFLEET-789, HYPERFLEET-012 |

[If release date is set and approaching:]
**Release date is [date] ([X] days away) with [Y] points remaining.**

---

### 5. Sprint Planning Priorities

Items ranked by suggested priority for the next sprint:

| Priority | Ticket | Summary | Pts | Why |
|----------|--------|---------|-----|-----|
| 1 | HYPERFLEET-456 | [Summary] | 3 | In progress — finish first |
| 2 | HYPERFLEET-345 | [Summary] | 5 | Blocked — needs unblocking |
| 3 | HYPERFLEET-123 | [Summary] | 5 | High priority, not in sprint |
| 4 | HYPERFLEET-789 | [Summary] | - | Medium priority, needs pointing |

---

### 6. Might These Belong to [Version]?

Items without a Fix Version that share components/labels with this release:

| Ticket | Summary | Type | Pts | Shared |
|--------|---------|------|-----|--------|
| HYPERFLEET-999 | [Summary] | Bug | 3 | component: platform |

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
- **sprint-report** / **sprint-status**: Complementary — those track the active sprint, this tracks the release
- **backlog-hygiene**: Complementary — backlog-hygiene audits the full backlog, this focuses on one release version
- **sprint-hygiene**: Complementary — sprint-hygiene audits sprint ticket quality, this audits release scope

## Notes

- Apply the ghostwriter skill for tone
- Keep the report scannable — tables over prose
- This skill surfaces candidates and priorities — it does NOT recommend a sprint commitment total
- If the release has many Done items, keep that section compact (count + points, not a full list)
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
