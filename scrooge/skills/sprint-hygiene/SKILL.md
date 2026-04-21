---
name: sprint-hygiene
description: Audit sprint tickets for hygiene - required fields, components, and duplicate
  detection
allowed-tools:
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_get_project_versions
  - mcp__atlassian__jira_search_fields
argument-hint: [scope: sprint|backlog|all]
disable-model-invocation: true
---

# Sprint Hygiene Check

Audit JIRA tickets for sprint readiness, including required fields, valid components, and potential duplicates. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Arguments
- `$1` (optional): Scope of check
  - `sprint` (default): Current sprint tickets only
  - `backlog`: Backlog items
  - `all`: All recent tickets

## Story Points Field Mapping

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen / Jira Software field — check this first |
| `customfield_10028` | Story Points | Classic field |

**Important:** These custom fields are NOT returned by default by MCP tools. When calling `mcp__atlassian__jira_get_sprint_issues`, `mcp__atlassian__jira_search`, or `mcp__atlassian__jira_get_issue`, you **must** include them in the `fields` parameter:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,fixVersions,customfield_10016,customfield_10028,customfield_10464"
```

## Activity Type Field

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10464` | Activity Type | Select dropdown |

## Behavior

### 1. Get Tickets to Audit

**Sprint scope (default):**
- Use `mcp__atlassian__jira_get_agile_boards` to find the board for project HYPERFLEET
- Use `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint
- Use `mcp__atlassian__jira_get_sprint_issues` to pull all sprint issues

**Backlog scope:**
- Use `mcp__atlassian__jira_search` with JQL: `project = HYPERFLEET AND status != Done AND status != Closed AND (sprint not in openSprints() OR sprint is EMPTY) ORDER BY updated ASC`

### 2. Check for Potential Duplicates (CRITICAL)

For each ticket, use `mcp__atlassian__jira_search` to search for similar titles:
```
project = HYPERFLEET AND status != Done AND summary ~ "keyword"
```
Run for each ticket being checked, using key terms from the summary.

### 3. Field Completeness Checks

For each ticket (using data from sprint issues or `mcp__atlassian__jira_get_issue` for full details):

- **Missing Story Points**: Check `customfield_10016` and `customfield_10028` — flag if both are empty (for Stories, Tasks, Bugs)
- **Missing/Inadequate Description**: Check description field exists and has > 50 characters
- **Missing Components**: Check components array — validate against `mcp__atlassian__jira_get_project_components`
- **Missing Activity Type**: Check `customfield_10464` is set
- **Stale Tickets**: Flag issues with `updated` older than 7 days
- **Missing Labels**: Check labels array

### 4. Individual Ticket Inspection

For tickets needing deeper checks, use `mcp__atlassian__jira_get_issue` to verify:
- **Title**: Clear, actionable, under 100 characters
- **Acceptance Criteria**: At least 2 clear, testable criteria in description
- **No ambiguous language**: Check for "maybe", "probably", "TBD", "possibly"

## Output Format

### Sprint Hygiene Report

#### CRITICAL: Potential Duplicates
| Ticket | Summary | Similar To | Similarity |
|--------|---------|------------|------------|
| TICKET-1 | [Summary] | TICKET-X | High/Medium |

**Action Required:** Review and close duplicates or link as related.

---

#### Missing Story Points
| Ticket | Summary | Type |
|--------|---------|------|
| TICKET-1 | [Summary] | Story |

**Action Required:** These tickets need estimation before sprint planning.

---

#### Missing/Inadequate Description
| Ticket | Summary | Description Length |
|--------|---------|-------------------|
| TICKET-1 | [Summary] | Empty |
| TICKET-2 | [Summary] | < 50 chars |

**Action Required:** Add detailed description with context and acceptance criteria.

---

#### Invalid or Missing Components
| Ticket | Summary | Current Component |
|--------|---------|-------------------|
| TICKET-1 | [Summary] | None |
| TICKET-2 | [Summary] | InvalidComponent |

**Valid Components:** Verify against `mcp__atlassian__jira_get_project_components`.

**Action Required:** Assign valid component for tracking.

---

#### Missing Activity Type
| Ticket | Summary | Type |
|--------|---------|------|
| TICKET-1 | [Summary] | Story |

**Valid Activity Types:** Associate Wellness & Development, Incidents & Support, Security & Compliance, Quality / Stability / Reliability, Future Sustainability, Product / Portfolio Work

**Action Required:** Set Activity Type for capacity planning.

---

#### Missing Fix Version
| Ticket | Summary | Type | Priority |
|--------|---------|------|----------|
| TICKET-1 | [Summary] | Story | High |

**Note:** Fix Version is informational, not a required field. Items without a Fix Version may not be tied to a specific release. Use `mcp__atlassian__jira_get_project_versions` to see available versions.

---

#### Stale Tickets (No Update 7+ Days)
| Ticket | Summary | Status | Last Updated |
|--------|---------|--------|--------------|

**Action Required:** Update status or add comment on progress.

---

### Summary Score (6 Required Fields)

| Check | Pass | Fail | Score |
|-------|------|------|-------|
| Title | X | X | X% |
| Description | X | X | X% |
| Acceptance Criteria | X | X | X% |
| Story Points | X | X | X% |
| Component (valid) | X | X | X% |
| Activity Type | X | X | X% |
| **Overall** | | | **X%** |

### Quality Checks

| Check | Pass | Fail |
|-------|------|------|
| **Not Duplicate (CRITICAL)** | X | X |
| No Ambiguous Language | X | X |
| Freshness (updated < 7d) | X | X |

### Sprint Readiness
- **Ready for Sprint:** X tickets
- **Needs Work:** X tickets
- **Critical Issues:** X tickets

### Top Priority Fixes
1. [Most critical hygiene issue - duplicates are highest priority]
2. [Second priority]
3. [Third priority]

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
