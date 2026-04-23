---
name: create-feature
description: Creates a JIRA Feature with problem statement, benefit hypothesis, and
  child epic breakdown. Activates when users ask to create a feature or initiative.
allowed-tools:
  - mcp__atlassian__jira_create_issue
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_search
---

# JIRA Feature Creator

Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Important: Feature-Specific Notes

- Features sit ABOVE epics in the JIRA hierarchy (Feature > Epic > Story)
- Features do NOT get story points (they are planning-level artifacts)
- Features do NOT get activity types (set on child epics/stories)
- Acceptance criteria should be outcome-based, not task-based
- If `issue_type: Feature` fails, try `issue_type: Initiative` (JIRA instance-specific)

## Writing Style

When writing descriptions, apply the ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create a feature", "I need a feature for..."
- "create a portfolio item", "create an initiative"
- "top-level feature for...", "create a feature to group these epics"

## Description Format

Write descriptions in **Markdown**. The MCP server converts to JIRA's native format automatically.

- NO curly braces `{}` in content -- they break JIRA rendering. Use `:id` or SCREAMING_CASE instead.
- API endpoints: `POST /api/v1/clusters/:id` (colon notation, never `/clusters/{id}`)

## Workflow

### Step 1: Gather Requirements

Ask the user:
- What problem are we solving? (Problem Statement)
- What outcome do we expect? (Benefit Hypothesis)
- How will we measure success? (Key Metrics)
- What's the high-level approach? (Solution Overview)
- What's in scope vs out of scope?
- What epics does this contain?

### Step 2: Discover Valid Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to check available components.

### Step 3: Create the Feature

Use `mcp__atlassian__jira_create_issue` with:
- `project_key`: `HYPERFLEET`
- `summary`: `Feature: Title`
- `issue_type`: `Feature`
- `description`: The feature description in Markdown (see template below)
- `components`: component name (if applicable)
- `additional_fields`: JSON string with custom fields:
  ```json
  {
    "priority": {"name": "High"}
  }
  ```

Notes:
- No story points -- features are planning-level artifacts
- No activity type -- set on child epics/stories instead
- If `Feature` type returns an error, try `Initiative` instead

### Description Template

```markdown
# Feature Title

### Problem Statement

Our [system/service/platform] is intended to [goals it should achieve].
We have observed that [what isn't working or what gap exists],
which is causing [adverse effect on teams, reliability, velocity, or cost].

### Benefit Hypothesis

We believe [measurable outcome]
will be achieved if [target users/teams]
can successfully [user outcome or capability]
using [this feature].

**Key Metrics:**
- [Metric 1]: current [baseline], target [goal]
- [Metric 2]: current [baseline], target [goal]

### Solution Overview

[2-4 sentences: high-level approach. What are we building or changing?
Keep this at the level a skip-level manager could understand.]

**Architectural Impact:**
- Systems affected: [list of services/components]
- New dependencies: [or "None"]
- Operational changes: [e.g., new on-call responsibilities, or "None"]

### Scope

**In Scope:**
- [Deliverable 1]
- [Deliverable 2]
- [Deliverable 3]

**Out of Scope:**
- [Item 1] (reason for deferral)
- [Item 2] (reason for deferral)

### Acceptance Criteria

- [Observable outcome 1 -- measurable system state, not a task]
- [Observable outcome 2]
- [Observable outcome 3]
- [Observable outcome 4]

### Child Epics

- [HYPERFLEET-XXX]: [Epic title] (estimated XX points)
- [HYPERFLEET-XXX]: [Epic title] (estimated XX points)
- [HYPERFLEET-XXX]: [Epic title] (estimated XX points)

### Dependencies

- Depends on: [FEATURE/EPIC-XXX or "None"]
- Blocks: [FEATURE/EPIC-XXX or "None"]
- External: [e.g., "Security review", "Quay.io org provisioning", or "None"]

### Risks

- [Risk 1] (likelihood: H/M/L, impact: H/M/L): [mitigation]
- [Risk 2] (likelihood: H/M/L, impact: H/M/L): [mitigation]

### Open Questions

- [Decision that must be resolved before or during implementation]
- [Question]
```

### Step 4: Post-Creation

Use `mcp__atlassian__jira_get_issue` to verify the ticket was created with all fields.

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Feature
Summary: [Title]
Priority: [Priority]
Link: https://redhat.atlassian.net/browse/HYPERFLEET-XXX

Manual steps needed:
- Link child epics to this feature
- Add labels [if applicable]
```

## Troubleshooting

### Issue Type Not Recognised
Some JIRA instances use `Initiative` instead of `Feature`. If `Feature` fails, retry with `issue_type: Initiative`.

## Integration

- **create-epic**: Create child epics after feature is set up
- **hygiene**: Validate feature quality

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
