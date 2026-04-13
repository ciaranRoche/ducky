---
name: new-tickets
description: Daily scan of tickets created in the last 24 hours — grooming quality
  assessment.
allowed-tools:
  - mcp__atlassian__search
  - mcp__atlassian__get_issue
  - mcp__atlassian__get_project_components
---

# New Tickets: Daily Grooming Quality Check

Scan for tickets created in the last 24 hours and assess their grooming quality. Read-only — surfaces issues, never modifies tickets. Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Behavior

### 1. Find New Tickets

Search using `mcp__atlassian__search` with JQL:
```
project = HYPERFLEET AND created >= -24h ORDER BY created DESC
```

If no tickets found, report that and stop.

### 2. Assess Each Ticket

For each new ticket, use `mcp__atlassian__get_issue` to read full details, then check:

**Required fields:**
- Summary (not generic like "Bug" or "Fix thing")
- Description (exists and has meaningful content, not just a title repeat)
- Issue type (Bug, Story, Task, Epic)
- Priority set (not just default)

**Grooming quality by type:**

For **Bugs:**
- Steps to reproduce present
- Expected vs actual behavior described
- Environment/version noted

For **Stories:**
- User story format or clear problem statement
- Acceptance criteria present
- Reasonable scope (not an epic disguised as a story)

For **Tasks:**
- Clear definition of done
- Not a duplicate of an existing ticket

For **Epics:**
- Success criteria defined
- Has or references child stories

**General quality:**
- Story points assigned
- Component set (verify against `mcp__atlassian__get_project_components`)
- Labels present
- Sprint assigned

### 3. Score and Classify

Rate each ticket:
- **Ready**: all required fields present, good grooming quality
- **Needs work**: missing 1-2 fields or quality gaps
- **Incomplete**: missing description, acceptance criteria, or other critical content

## Output Format

```
## New Tickets Report
**[X] tickets created in the last 24 hours**

### Needs Attention
| Ticket | Type | Summary | Issues |
|--------|------|---------|--------|
| [PROJ-123] | Bug | Login fails | No steps to reproduce, no priority |
| [PROJ-456] | Story | Add feature X | Missing acceptance criteria |

### Ready for Sprint
| Ticket | Type | Summary | Points |
|--------|------|---------|--------|
| [PROJ-789] | Task | Update docs | 2 |

### Summary
- Ready: X tickets
- Needs work: X tickets
- Incomplete: X tickets
```

## Notes

- Apply the ghostwriter skill for tone
- Be specific about what's missing — "missing acceptance criteria" not "needs grooming"
- Don't flag minor style preferences, focus on substantive gaps
- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
