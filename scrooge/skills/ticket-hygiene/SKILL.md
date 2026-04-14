---
name: ticket-hygiene
description: Validates JIRA tickets have required fields and quality standards. Mechanical
  field checks for sprint readiness.
allowed-tools:
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_search_fields
  - mcp__atlassian__jira_search
---

# JIRA Ticket Hygiene Check

Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Story Points Field Mapping

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen / Jira Software field — check this first |
| `customfield_10028` | Story Points | Classic field |

## Activity Type Field

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10464` | Activity Type | Select dropdown |

## When to Use This Skill

Activate when the user:
- Asks for a "hygiene check" on a ticket
- Asks to "validate fields" on a ticket
- Asks "does this ticket have all the required fields?"
- Wants a quick field completeness check

## Hygiene Checklist

### Required Fields (Must Have)
| Field | Requirement |
|-------|-------------|
| Title | Clear, actionable, under 100 characters |
| Description | Detailed context (recommend > 100 characters) |
| Acceptance Criteria | At least 2 clear, testable criteria |
| Story Points | Set (scale: 0, 1, 3, 5, 8, 13) |
| Component | Set to a valid project component |
| Activity Type | Set for capacity planning |

### Recommended Fields
| Field | Requirement |
|-------|-------------|
| Labels | At least 1 relevant label |
| Epic Link | Connected to parent epic (for Stories) |
| Fix Version | Target release identified |
| Priority | Explicitly set (not just default) |

### Quality Checks
- **CRITICAL: Not a duplicate** - Search for similar titles/descriptions in backlog before adding
- No ambiguous language ("maybe", "probably", "TBD", "possibly")
- Technical approach outlined or referenced
- Dependencies identified and linked
- Scope is achievable in one sprint

## How to Check a Ticket

### Step 1: Fetch Ticket Details

Use `mcp__atlassian__jira_get_issue` with the ticket key. The response includes all fields in structured JSON.

### Step 2: Validate Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to get the list of valid components. Check that the ticket's component is in this list.

## Output Format

### Ticket: TICKET-KEY

**Summary:** [Ticket title]

#### Hygiene Assessment

| Check | Status | Notes |
|-------|--------|-------|
| Title | PASS/FAIL | [Issue if any] |
| Description | PASS/FAIL | [Length: X chars] |
| Acceptance Criteria | PASS/FAIL | [Count: X criteria] |
| Story Points | PASS/FAIL | [Value or "Missing"] |
| Component | PASS/FAIL | [Must be a valid project component] |
| Activity Type | PASS/FAIL | [Type or "Uncategorized"] |

#### Overall Score: X/6 Required Checks Passed

#### Verdict
- **ALL CLEAR** - All required fields present, good quality
- **NEEDS MINOR FIXES** - 1-2 issues to address
- **NOT READY** - Multiple critical issues

#### Recommended Actions
1. [Specific action to fix issue 1]
2. [Specific action to fix issue 2]

## Activity Types (Sankey Capacity Allocation)

Activity Type is **required** for sprint/kanban capacity planning. Tickets without an Activity Type appear as "Uncategorized" and cannot be properly allocated.

### Reactive Work (Non-Negotiable First)
| Activity Type | Description | Examples |
|---------------|-------------|----------|
| **Associate Wellness & Development** | Onboarding, team growth, training, associate experience | Training sessions, mentorship |
| **Incidents & Support** | Escalations, production issues | Customer escalations, outages |
| **Security & Compliance** | Vulnerabilities and weaknesses, CVEs | Security patches, compliance fixes |

### Core Principles (Quality Focus)
| Activity Type | Description | Examples |
|---------------|-------------|----------|
| **Quality / Stability / Reliability** | Bugs, SLOs, chores, tech debt, PMR action items, toil reduction | Bug fixes, performance improvements |

### Proactive Work (Balance Remaining Capacity)
| Activity Type | Description | Examples |
|---------------|-------------|----------|
| **Future Sustainability** | Productivity improvements, team improvements, upstream, proactive architecture, enablement | Tooling, automation, refactoring |
| **Product / Portfolio Work** | Strategic portfolio (HATSTRAT), strategic product, product outcome, BU features | New features, product enhancements |

### Priority Order
1. **Non-Negotiable**: Achieve SLAs for Escalations & CVEs
2. **Core Principles**: Reduce bug backlog, ensure quality/stability/reliability
3. **Then Balance**: Set up for long-term success by balancing remaining capacity between Future Sustainability and Product Work

## Red Flags to Highlight

- Descriptions under 50 characters
- "TBD" or placeholder text in any field
- Story points of 13+ (must be broken down)
- No acceptance criteria at all
- Vague titles like "Fix bug" or "Update feature"
- Tickets open > 30 days without progress
- **Missing Activity Type** (appears as Uncategorized in capacity planning)
- **Invalid Component** (must be a valid component for the configured project)

## Integration

- **sprint-hygiene**: Bulk audit of sprint tickets
- **ticket-triage**: Interactive deep-dive on ticket validity and sprint-readiness
- **set-activity-type**: Set or change activity type when missing

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
