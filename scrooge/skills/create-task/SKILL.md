---
name: create-task
description: Creates a JIRA Task for tech debt, infrastructure, documentation, or
  spike work. Activates when users ask to create a task, spike, tech debt ticket, or
  infrastructure work.
allowed-tools:
  - mcp__atlassian__jira_create_issue
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_link_to_epic
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_search_fields
---

# JIRA Task Creator

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

- "create a task", "add a task"
- "create a spike for...", "I need a spike", "investigate..."
- "tech debt ticket for...", "create a tech debt item"
- "infrastructure task", "documentation task"

## Description Format

Write descriptions in **Markdown**. The MCP server converts to JIRA's native format automatically.

- NO curly braces `{}` in content -- they break JIRA rendering. Use `:id` or SCREAMING_CASE instead.
- API endpoints: `POST /api/v1/clusters/:id` (colon notation, never `/clusters/{id}`)

## Task Sub-Types

Detect the sub-type from the user's request:
- **Spike/Investigation**: User says "spike", "investigate", "research", "explore options"
- **Tech Debt/Refactor**: User says "tech debt", "refactor", "clean up", "improve"
- **General**: Infrastructure, documentation, configuration, or anything else

Use the appropriate template variant below.

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
- Why does this matter? (Why)
- Is this a spike? (determines template)
- How complex is this? (Story Points)

### Step 2: Discover Valid Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to check available components.

### Step 3: Create the Task

Use `mcp__atlassian__jira_create_issue` with:
- `project_key`: `HYPERFLEET`
- `summary`: `Task title (< 100 chars)` — for spikes, prefix with `[SPIKE]`
- `issue_type`: `Task`
- `description`: The task description in Markdown (see templates below)
- `components`: component name (if applicable)
- `additional_fields`: JSON string with custom fields:
  ```json
  {
    "priority": {"name": "Normal"},
    "customfield_10016": 3,
    "customfield_10464": {"value": "Future Sustainability"}
  }
  ```

### Default Template (tech debt / infra / docs)

```markdown
### What

[What needs to be done. 2-3 sentences.]

### Why

- [Business or technical impact]
- [What problem this solves]

### Current State

[What is problematic now -- the pain point or gap.]

### Desired State

[What we want instead -- the target outcome.]

### Acceptance Criteria

- [Specific, testable criterion 1]
- [Criterion 2]
- [Criterion 3]

### Technical Approach

- [How this will be accomplished]
- Files/components affected:
  - `component-1`
  - `component-2`
```

### Spike Template (investigations / research / PoCs)

```markdown
### Research Question

[What we need to answer. Be specific.]

### Why

- [Why this investigation matters]
- [What decisions depend on the outcome]

### Time Box

[Max time to spend, e.g., "3 days" or "1 sprint"]

### Deliverable

[What the output should be: report, recommendation, PoC, ADR, etc.]

### Acceptance Criteria

- Research questions answered with evidence
- Deliverable produced and shared with team
- Recommendation documented with trade-offs
- Next steps identified
```

**Activity type** defaults to "Future Sustainability". Override when needed:
- Refactoring/quality work: "Quality / Stability / Reliability"
- Security tasks: "Security & Compliance"
- Customer-driven: "Incidents & Support"

Valid activity types: `Associate Wellness & Development`, `Incidents & Support`, `Security & Compliance`, `Quality / Stability / Reliability`, `Future Sustainability`, `Product / Portfolio Work`

### Step 4: Post-Creation

Use `mcp__atlassian__jira_get_issue` to verify the ticket was created with all fields.

If the task belongs to an epic, use `mcp__atlassian__jira_link_to_epic` to link it:
- `issue_key`: the new ticket key
- `epic_key`: the parent epic key

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Task
Summary: [Title]
Points: [X]
Priority: [Priority]
Activity Type: [Type]
Link: https://redhat.atlassian.net/browse/HYPERFLEET-XXX

Manual steps needed:
- Add labels [if applicable]
```

## Integration

- **story-pointer**: Estimate task complexity
- **ticket-hygiene**: Validate ticket quality after creation
- **ticket-triage**: Interactive triage session for deeper assessment

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
