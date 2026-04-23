---
name: set-activity-type
description: Sets or changes the Activity Type on an existing JIRA ticket for capacity
  planning. Activates when users ask to set, change, or add activity type to a ticket.
allowed-tools:
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_update_issue
  - mcp__atlassian__jira_search_fields
argument-hint: <ticket-key> [activity-type]
---

# JIRA Activity Type Setter

Uses `mcp__atlassian__*` MCP tools exclusively (not jira-cli).

## Activity Type Field

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10464` | Activity Type | Select dropdown — set as `{"customfield_10464": {"value": "Type Name"}}` |

## When to Use This Skill

- "set activity type on TICKET-KEY"
- "change the activity type for TICKET-KEY"
- "add activity type to TICKET-KEY"
- "what activity type should TICKET-KEY be?"
- "this ticket needs an activity type"
- "categorize TICKET-KEY"

## Activity Types (Sankey Capacity Allocation)

Activity Type is **required** for sprint/kanban capacity planning. Tickets without an Activity Type appear as "Uncategorized" and cannot be properly allocated.

### Reactive Work (Non-Negotiable First)
| Activity Type | Description | Examples |
|---------------|-------------|----------|
| **Associate Wellness & Development** | Onboarding, team growth, training, associate experience | Training sessions, mentorship, onboarding |
| **Incidents & Support** | Escalations, production issues | Customer escalations, outages, support tickets |
| **Security & Compliance** | Vulnerabilities and weaknesses, CVEs | Security patches, compliance fixes, audit items |

### Core Principles (Quality Focus)
| Activity Type | Description | Examples |
|---------------|-------------|----------|
| **Quality / Stability / Reliability** | Bugs, SLOs, chores, tech debt, PMR action items, toil reduction | Bug fixes, performance improvements, reliability work |

### Proactive Work (Balance Remaining Capacity)
| Activity Type | Description | Examples |
|---------------|-------------|----------|
| **Future Sustainability** | Productivity improvements, team improvements, upstream, proactive architecture, enablement | Tooling, automation, refactoring, infrastructure |
| **Product / Portfolio Work** | Strategic portfolio (HATSTRAT), strategic product, product outcome, BU features | New features, product enhancements |

### Priority Order
1. **Non-Negotiable**: Achieve SLAs for Escalations & CVEs
2. **Core Principles**: Reduce bug backlog, ensure quality/stability/reliability
3. **Then Balance**: Set up for long-term success by balancing remaining capacity between Future Sustainability and Product Work

## Workflow

### Step 1: Fetch Ticket Details

Use `mcp__atlassian__jira_get_issue` with the ticket key to review the ticket's summary, type, description, and current fields.

### Step 2: Determine Activity Type

If the user provided an activity type, validate it against the list above.

If not provided, recommend one based on the ticket's content:

| Ticket Signals | Recommended Activity Type |
|----------------|--------------------------|
| Bug, defect, SLO, reliability, tech debt, toil | Quality / Stability / Reliability |
| New feature, product requirement, enhancement, HATSTRAT | Product / Portfolio Work |
| Refactoring, tooling, automation, infrastructure, enablement | Future Sustainability |
| CVE, vulnerability, compliance, audit | Security & Compliance |
| Escalation, outage, support, customer issue | Incidents & Support |
| Training, onboarding, mentorship, team growth | Associate Wellness & Development |

Present your recommendation with reasoning and ask the user to confirm before setting.

### Step 3: Set Activity Type

Use `mcp__atlassian__jira_update_issue`:
- `issue_key`: the ticket key
- `fields`: `{"customfield_10464": {"value": "Activity Type Value"}}`

### Step 4: Verify

Use `mcp__atlassian__jira_get_issue` to confirm the field was set correctly.

## Output Format

```
Activity Type Updated: TICKET-KEY

Summary: [Ticket title]
Activity Type: [Old value] -> [New value]
Reasoning: [Why this category fits]
Link: https://redhat.atlassian.net/browse/TICKET-KEY
```

## Bulk Mode

If the user asks to set activity types for multiple tickets or a sprint's worth of tickets, find tickets missing activity type first:

Use `mcp__atlassian__jira_search` with JQL:
```
project = HYPERFLEET AND "Activity Type" is EMPTY AND sprint in openSprints()
```

Then process each ticket individually, recommending and confirming with the user before setting.

## Integration

- **hygiene**: Activity Type is one of the 6 required field checks (both single-ticket and sprint audit)
- **ticket-triage**: Interactive triage may identify missing Activity Type
- **create-story / create-bug / create-task**: Set Activity Type at creation time

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
