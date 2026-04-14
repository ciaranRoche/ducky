---
name: create-story
description: Creates a JIRA Story with user story format, acceptance criteria, and
  technical notes. Activates when users ask to create a story or user story.
allowed-tools:
  - mcp__atlassian__jira_create_issue
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_link_to_epic
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_search_fields
---

# JIRA Story Creator

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

- "create a story", "write a story", "add a user story"
- "I need a story for..."
- "create a story for this feature"

## Description Format

Write descriptions in **Markdown**. The MCP server converts to JIRA's native format automatically.

- NO curly braces `{}` in content -- they break JIRA rendering. Use `:id` or SCREAMING_CASE instead.
- API endpoints: `POST /api/v1/clusters/:id` (colon notation, never `/clusters/{id}`)

## Story Format

Choose the format that best fits:

**Standard** (most common):
```
As a [role], I want [capability], so that [benefit].
```

**Technical** (for backend/API/infrastructure work):
```
[Action] the [result] for [object]
Example: "Validate the adapter status contract before storing in database"
```

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
- What needs to be done? (What)
- Who benefits and why? (Why)
- How will we know it's done? (Acceptance Criteria)
- How complex is this? (Story Points)

### Step 2: Discover Valid Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to check available components.

### Step 3: Create the Story

Use `mcp__atlassian__jira_create_issue` with:
- `project_key`: `HYPERFLEET`
- `summary`: `Story title (< 100 chars)`
- `issue_type`: `Story`
- `description`: The story description in Markdown (see template below)
- `components`: component name (if applicable)
- `additional_fields`: JSON string with custom fields:
  ```json
  {
    "priority": {"name": "Normal"},
    "customfield_10016": 5,
    "customfield_10464": {"value": "Product / Portfolio Work"}
  }
  ```

### Description Template

```markdown
### What

[User story or technical description. 2-4 sentences.]

### Why

- [Reason 1 -- who benefits and how]
- [Reason 2 -- what problem it solves]

### Acceptance Criteria

- [Given/When/Then or testable criterion 1]
- [Criterion 2]
- [Criterion 3]

### Technical Notes

- Implementation approach: [brief description]
- Files/components affected:
  - `component-1`
  - `component-2`
- API/DB changes: [if any, or "None"]

### Out of Scope

- [Explicit exclusion 1]
```

**Activity type** defaults to "Product / Portfolio Work". Override when needed:
- Infrastructure stories: "Future Sustainability"
- Reliability/quality stories: "Quality / Stability / Reliability"
- Security work: "Security & Compliance"

Valid activity types: `Associate Wellness & Development`, `Incidents & Support`, `Security & Compliance`, `Quality / Stability / Reliability`, `Future Sustainability`, `Product / Portfolio Work`

### Step 4: Post-Creation

Use `mcp__atlassian__jira_get_issue` to verify the ticket was created with all fields.

If the story belongs to an epic, use `mcp__atlassian__jira_link_to_epic` to link it:
- `issue_key`: the new ticket key
- `epic_key`: the parent epic key

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Story
Summary: [Title]
Points: [X]
Priority: [Priority]
Activity Type: [Type]
Link: https://redhat.atlassian.net/browse/HYPERFLEET-XXX

Manual steps needed:
- Add labels [if applicable]
```

## Integration

- **story-pointer**: Detailed estimation methodology
- **ticket-hygiene**: Validate ticket quality after creation
- **ticket-triage**: Interactive triage session for deeper assessment

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
