---
name: create-bug
description: Creates a JIRA Bug with steps to reproduce, expected/actual behavior,
  and fix criteria. Activates when users ask to create a bug report, file a bug, or
  report an issue.
allowed-tools:
  - mcp__atlassian__jira_create_issue
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_link_to_epic
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_search_fields
---

# JIRA Bug Creator

Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Custom Field Reference

| Field ID | Name | Format |
|----------|------|--------|
| `customfield_10016` | Story point estimate | Number (0, 1, 3, 5, 8, 13) |
| `customfield_10464` | Activity Type | `{"value": "Type Name"}` |

## Writing Style

When writing descriptions, apply the ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create a bug", "file a bug", "report a bug"
- "there's a bug in...", "I found a bug"
- "this is broken", "create a bug report"

## Description Format

Write descriptions in **Markdown**. The MCP server converts to JIRA's native format automatically.

- NO curly braces `{}` in content -- they break JIRA rendering (learned from HYPERFLEET-258 where `{customer-id}` broke the ticket). Use `:id` or SCREAMING_CASE instead.
- API endpoints: `POST /api/v1/clusters/:id` (colon notation, never `/clusters/{id}`)

## Priority Guidance

- `Blocker`: Blocks development/testing, must be fixed immediately
- `Critical`: Crashes, data loss, severe memory leak
- `Major`: Major loss of function (default for bugs)
- `Normal`: Minor functional issue
- `Minor`: Cosmetic, easy workaround

## Story Points

Use scale: 0, 1, 3, 5, 8, 13
- 0: Tracking only (negligible effort)
- 1: Trivial (< half day)
- 3: Straightforward (1-2 days)
- 5: Medium complexity (2-4 days)
- 8: Large, may need design doc (1+ week)
- 13: Too large -- break it down

For detailed estimation, reference the **story-pointer** skill.

## Workflow

### Step 1: Gather Requirements

Ask the user if needed:
- What is the bug? (What)
- How do you reproduce it? (Steps)
- What should happen vs what actually happens?
- How severe is it? (Priority)

### Step 2: Discover Valid Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to check available components.

### Step 3: Create the Bug

Use `mcp__atlassian__jira_create_issue` with:
- `project_key`: `HYPERFLEET`
- `summary`: `Bug: Brief description (< 100 chars)`
- `issue_type`: `Bug`
- `description`: The bug description in Markdown (see template below)
- `components`: component name (if applicable)
- `additional_fields`: JSON string with custom fields:
  ```json
  {
    "priority": {"name": "Major"},
    "customfield_10016": 5,
    "customfield_10464": {"value": "Quality / Stability / Reliability"}
  }
  ```

### Description Template

```markdown
### What

[Description of the bug and its impact. 2-3 sentences.]

### Steps to Reproduce

1. Step 1
2. Step 2
3. Step 3

### Expected Behavior

[What should happen when following the steps above.]

### Actual Behavior

[What actually happens. Include error messages if available.]

### Why

- [Who is affected -- users, teams, system reliability]
- [Severity of impact -- data loss, degraded performance, blocking work]

### Acceptance Criteria

- Root cause identified and fixed
- Regression test added covering this scenario
- No related side effects introduced
- [Additional criterion specific to this bug]

### Technical Notes

- Affected component: `component-name`
- Error location: `file/path`
- Relevant logs/errors: [describe, add screenshots via web UI]
```

**Activity type** defaults to "Quality / Stability / Reliability". Override for:
- Security bugs: "Security & Compliance"
- Customer-reported: "Incidents & Support"

Valid activity types: `Associate Wellness & Development`, `Incidents & Support`, `Security & Compliance`, `Quality / Stability / Reliability`, `Future Sustainability`, `Product / Portfolio Work`

### Step 4: Post-Creation

Use `mcp__atlassian__jira_get_issue` to verify the ticket was created with all fields.

If the bug belongs to an epic, use `mcp__atlassian__jira_link_to_epic` to link it:
- `issue_key`: the new ticket key
- `epic_key`: the parent epic key

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Bug
Summary: [Title]
Points: [X]
Priority: [Priority]
Activity Type: Quality / Stability / Reliability
Link: https://redhat.atlassian.net/browse/HYPERFLEET-XXX

Manual steps needed:
- Add screenshots/logs [if available]
- Add labels [if applicable]
```

## Integration

- **story-pointer**: Estimate complexity of the fix
- **hygiene**: Validate ticket quality after creation
- **ticket-triage**: Interactive triage session for deeper assessment

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
