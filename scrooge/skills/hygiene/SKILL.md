---
name: hygiene
description: Validate JIRA ticket fields and quality standards. Run on a single ticket
  or bulk-audit all sprint tickets. Activates when users ask to "check hygiene",
  "validate fields", or "audit sprint tickets".
allowed-tools:
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_get_project_versions
  - mcp__atlassian__jira_search_fields
  - mcp__atlassian__jira_get_agile_boards
  - mcp__atlassian__jira_get_sprints_from_board
  - mcp__atlassian__jira_get_sprint_issues
argument-hint: <ticket-key> | sprint
---

# JIRA Hygiene Check

Validate JIRA tickets for field completeness, quality standards, and potential duplicates. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Arguments
- `$1`: Scope of the check
  - `<ticket-key>` (e.g., HYPERFLEET-123): Single ticket hygiene check
  - `sprint`: Bulk audit of all tickets in the active sprint

## Custom Field Reference

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen — check first |
| `customfield_10028` | Story Points | Classic fallback |
| `customfield_10464` | Activity Type | Select dropdown |

**Important:** These custom fields are NOT returned by default by MCP tools. You **must** include them in the `fields` parameter:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,fixVersions,customfield_10016,customfield_10028,customfield_10464"
```

## Required Field Checks (6-point score)

These checks apply in both single-ticket and sprint modes:

| # | Field | Requirement |
|---|-------|-------------|
| 1 | Title | Clear, actionable, under 100 characters |
| 2 | Description | Detailed context (> 100 characters) |
| 3 | Acceptance Criteria | At least 2 clear, testable criteria |
| 4 | Story Points | Set (scale: 0, 1, 3, 5, 8, 13) — check `customfield_10016` then `customfield_10028` |
| 5 | Component | Set to a valid project component |
| 6 | Activity Type | Set for capacity planning — check `customfield_10464` |

## Recommended Fields (informational, not scored)

| Field | Requirement |
|-------|-------------|
| Labels | At least 1 relevant label |
| Epic Link | Connected to parent epic (for Stories) |
| Fix Version | Target release identified |
| Priority | Explicitly set (not just default) |

## Quality Checks

- **CRITICAL: Not a duplicate** — search for similar titles/descriptions
- No ambiguous language ("maybe", "probably", "TBD", "possibly")
- Technical approach outlined or referenced
- Dependencies identified and linked
- Scope achievable in one sprint

## Red Flags

- Descriptions under 50 characters
- "TBD" or placeholder text in any field
- Story points of 13+ (must be broken down)
- No acceptance criteria at all
- Vague titles like "Fix bug" or "Update feature"
- Tickets open > 30 days without progress
- Missing Activity Type (appears as Uncategorized in capacity planning)
- Invalid component (must be a valid project component)

---

## Mode: Single Ticket (`hygiene TICKET-KEY`)

### Step 1: Fetch Ticket Details

Use `mcp__atlassian__jira_get_issue` with the ticket key. Pass the `fields` parameter.

### Step 2: Validate Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to verify the ticket's component is valid.

### Step 3: Check for Duplicates

Use `mcp__atlassian__jira_search` with key terms from the summary:
```
project = HYPERFLEET AND status != Done AND summary ~ "keyword"
```

### Output Format (Single Ticket)

```
### Ticket: TICKET-KEY

**Summary:** [Ticket title]

#### Hygiene Assessment

| Check | Status | Notes |
|-------|--------|-------|
| Title | PASS/FAIL | [Issue if any] |
| Description | PASS/FAIL | [Length: X chars] |
| Acceptance Criteria | PASS/FAIL | [Count: X criteria] |
| Story Points | PASS/FAIL | [Value or "Missing"] |
| Component | PASS/FAIL | [Must be valid project component] |
| Activity Type | PASS/FAIL | [Type or "Uncategorized"] |

#### Overall Score: X/6 Required Checks Passed

#### Recommended Fields
| Field | Status | Notes |
|-------|--------|-------|
| Fix Version | SET/MISSING | [Version name or "No target release"] |
| Labels | SET/MISSING | [Count or "None"] |
| Epic Link | SET/MISSING | [Epic key or "Not linked"] |
| Priority | SET/MISSING | [Value] |

#### Verdict
- **ALL CLEAR** — All required fields present, good quality
- **NEEDS MINOR FIXES** — 1-2 issues to address
- **NOT READY** — Multiple critical issues

#### Recommended Actions
1. [Specific action to fix issue 1]
2. [Specific action to fix issue 2]
```

---

## Mode: Sprint Audit (`hygiene sprint`)

### Step 1: Get Sprint Tickets

- Use `mcp__atlassian__jira_get_agile_boards` to find the board for project HYPERFLEET
- Use `mcp__atlassian__jira_get_sprints_from_board` to find the active sprint
- Use `mcp__atlassian__jira_get_sprint_issues` to pull all sprint issues (pass `fields` parameter)

### Step 2: Check for Duplicates (CRITICAL)

For each ticket, use `mcp__atlassian__jira_search` with key terms from the summary:
```
project = HYPERFLEET AND status != Done AND summary ~ "keyword"
```

### Step 3: Field Completeness

For each sprint ticket, run the 6 required field checks. Also flag:
- Missing/inadequate descriptions (< 50 chars)
- Invalid or missing components (validate against `mcp__atlassian__jira_get_project_components`)
- Missing Activity Type
- Missing Fix Version (informational)
- Stale tickets (no update in 7+ days)
- Missing labels

### Output Format (Sprint Audit)

```
## Sprint Hygiene Report

### CRITICAL: Potential Duplicates
| Ticket | Summary | Similar To | Similarity |
|--------|---------|------------|------------|
| TICKET-1 | [Summary] | TICKET-X | High/Medium |

**Action Required:** Review and close duplicates or link as related.

---

### Missing Story Points
| Ticket | Summary | Type |
|--------|---------|------|

**Action Required:** Estimate before sprint execution.

---

### Missing/Inadequate Description
| Ticket | Summary | Description Length |
|--------|---------|-------------------|

---

### Invalid or Missing Components
| Ticket | Summary | Current Component |
|--------|---------|-------------------|

---

### Missing Activity Type
| Ticket | Summary | Type |
|--------|---------|------|

---

### Missing Fix Version
| Ticket | Summary | Type | Priority |
|--------|---------|------|----------|

**Note:** Fix Version is informational, not required.

---

### Stale Tickets (No Update 7+ Days)
| Ticket | Summary | Status | Last Updated |
|--------|---------|--------|--------------|

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

### Sprint Readiness
- **Ready:** X tickets
- **Needs Work:** X tickets
- **Critical Issues:** X tickets

### Top Priority Fixes
1. [Most critical — duplicates are highest priority]
2. [Second priority]
3. [Third priority]
```

## Activity Types (Sankey Capacity Allocation)

Activity Type is **required** for sprint/kanban capacity planning.

| Category | Activity Type | Examples |
|----------|---------------|----------|
| Reactive (non-negotiable) | Associate Wellness & Development | Training, mentorship, onboarding |
| Reactive (non-negotiable) | Incidents & Support | Escalations, outages, customer issues |
| Reactive (non-negotiable) | Security & Compliance | CVEs, vulnerabilities, compliance fixes |
| Core Principles | Quality / Stability / Reliability | Bugs, SLOs, tech debt, toil reduction |
| Proactive | Future Sustainability | Tooling, automation, refactoring |
| Proactive | Product / Portfolio Work | New features, product enhancements |

## Integration

- **ticket-triage**: Deep interactive assessment (runs these checks inline as Step 2)
- **set-activity-type**: Set Activity Type when flagged as missing
- **story-pointer**: Estimate story points when flagged as missing

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
