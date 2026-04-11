---
name: set-activity-type
description: Sets or changes the Activity Type on an existing JIRA ticket for capacity planning. Activates when users ask to set, change, or add activity type to a ticket.
argument-hint: <ticket-key> [activity-type]
---

# JIRA Activity Type Setter

## Configuration

- `DUCKY_JIRA_PROJECT`: JIRA project key (default: `HYPERFLEET`)
- `JIRA_BASE_URL`: JIRA instance URL (default: `https://redhat.atlassian.net`)

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

```bash
jira issue view TICKET-KEY --plain 2>/dev/null
```

Review the ticket's summary, type, description, and current fields to understand what category of work it represents.

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

```bash
jira issue edit TICKET-KEY --custom activity-type="Activity Type Value" --no-input
```

### Step 4: Verify

```bash
jira issue view TICKET-KEY --plain 2>/dev/null
```

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

```bash
jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND 'Activity Type' is EMPTY AND sprint in openSprints()" --plain 2>/dev/null
```

Then process each ticket individually, recommending and confirming with the user before setting.

## Prerequisites

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`

## Integration

- **ticket-hygiene**: Activity Type is one of the 6 required field checks
- **sprint-hygiene**: Bulk audit flags tickets missing Activity Type
- **ticket-triage**: Interactive triage may identify missing Activity Type
- **create-story / create-bug / create-task**: Set Activity Type at creation time
