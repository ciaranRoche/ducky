---
name: create-epic
description: Creates a JIRA Epic with scope, success criteria, and child story breakdown.
  Activates when users ask to create an epic.
allowed-tools:
  - mcp__atlassian__jira_create_issue
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_search_fields
---

# JIRA Epic Creator

Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Custom Field Reference

| Field ID | Name | Format |
|----------|------|--------|
| `customfield_10011` | Epic Name | String (short name, required for epics) |

Epics do NOT get story points (they aggregate child story points) and do NOT get activity types (set on child stories instead).

## Writing Style

When writing descriptions, apply the ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create an epic", "I need an epic for..."
- "set up an epic for this feature area"
- "create an epic to track..."

## Description Format

Write descriptions in **Markdown**. The MCP server converts to JIRA's native format automatically.

- NO curly braces `{}` in content -- they break JIRA rendering. Use `:id` or SCREAMING_CASE instead.
- API endpoints: `POST /api/v1/clusters/:id` (colon notation, never `/clusters/{id}`)

## Workflow

### Step 1: Gather Requirements

Ask the user if needed:
- What are we building? (What)
- Why does this matter? (Why)
- What's in scope vs out of scope?
- What are the success criteria?
- What are the known risks or dependencies?

### Step 2: Discover Valid Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to check available components.

### Step 3: Create the Epic

Use `mcp__atlassian__jira_create_issue` with:
- `project_key`: `HYPERFLEET`
- `summary`: `Epic: Full Title Here`
- `issue_type`: `Epic`
- `description`: The epic description in Markdown (see template below)
- `components`: component name (if applicable)
- `additional_fields`: JSON string with custom fields:
  ```json
  {
    "customfield_10011": "Short Name"
  }
  ```

Notes:
- `customfield_10011` (Epic Name) is **required** for epics
- No `customfield_10016` -- epics aggregate child story points
- No `customfield_10464` -- activity type is set on child stories

### Description Template

```markdown
# Epic Title

### What

[What are we building? 2-3 sentences describing the deliverable.]

### Why

[Why does this matter? 1-2 sentences on the problem it solves or value it delivers.]

### Scope

**In Scope:**
- [Deliverable 1]
- [Deliverable 2]
- [Deliverable 3]

**Out of Scope:**
- [Item 1]
- [Item 2]

### Acceptance Criteria

- [Criterion 1 -- observable outcome]
- [Criterion 2]
- [Criterion 3]
- [Criterion 4]
- [Criterion 5]

### Dependencies

- Blocked by: [EPIC-XXX or "None"]
- Blocks: [EPIC-XXX or "None"]

### Risks

- [Risk 1]: [mitigation strategy]
- [Risk 2]: [mitigation strategy]

### Notes

[Additional context, links to design docs, related research.]
```

### Step 4: Post-Creation

Use `mcp__atlassian__jira_get_issue` to verify the ticket was created with all fields.

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Epic
Summary: [Title]
Epic Name: [Short Name]
Link: https://redhat.atlassian.net/browse/HYPERFLEET-XXX

Manual steps needed:
- Link to parent feature [if applicable]
- Add labels [if applicable]
- Create child stories
```

## Troubleshooting

### Epic Name Not Setting
Epic Name uses `customfield_10011` in `additional_fields`. Ensure it's passed as a plain string, not an object.

## Integration

- **create-story**: Create child stories after epic is set up
- **ticket-hygiene**: Validate epic quality

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
