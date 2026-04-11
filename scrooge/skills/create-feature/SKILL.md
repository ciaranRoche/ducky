---
name: create-feature
description: Creates a JIRA Feature with problem statement, benefit hypothesis, and child epic breakdown. Activates when users ask to create a feature or initiative.
---

# JIRA Feature Creator

## Configuration

- `DUCKY_JIRA_PROJECT`: JIRA project key (default: `HYPERFLEET`)
- `JIRA_BASE_URL`: JIRA instance URL (default: `https://redhat.atlassian.net`)

## Writing Style

When writing descriptions, apply the ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create a feature", "I need a feature for..."
- "create a portfolio item", "create an initiative"
- "top-level feature for...", "create a feature to group these epics"

## JIRA Wiki Markup (NOT Markdown)

- Headers: `h3. Title` (space after period, never `###`)
- Bullets: `* item`, nested: `** item` (never `-` or `•`)
- Bold: `*bold*`, Italic: `_italic_`
- Inline code: `{{code}}` (never backticks)
- NO curly braces `{}` in content -- they break JIRA rendering (learned from HYPERFLEET-258 where `{customer-id}` broke the ticket). Use `:id` or SCREAMING_CASE instead.
- API endpoints: `*POST* /api/v1/clusters/:id` (colon notation, never `/clusters/{id}`)
- NO code blocks via CLI (renders as empty gray box -- add manually in web UI)
- NO YAML comments in code blocks -- `#` is interpreted as `h1.` header
- Always write descriptions to a temp file, never inline strings

## Important: Feature-Specific Notes

- Features sit ABOVE epics in the JIRA hierarchy (Feature > Epic > Story)
- Features do NOT get story points (they are planning-level artifacts)
- Features do NOT get activity types (set on child epics/stories)
- Acceptance criteria should be outcome-based, not task-based
- If `--type Feature` fails, try `--type Initiative` (JIRA instance-specific)

## Workflow

### Step 1: Gather Requirements

Ask the user:
- What problem are we solving? (Problem Statement)
- What outcome do we expect? (Benefit Hypothesis)
- How will we measure success? (Key Metrics)
- What's the high-level approach? (Solution Overview)
- What's in scope vs out of scope?
- What epics does this contain?

### Step 2: Create Description File

```bash
cat > /tmp/feature-description.txt << 'EOF'
h1. Feature Title

h3. Problem Statement

Our [system/service/platform] is intended to [goals it should achieve].
We have observed that [what isn't working or what gap exists],
which is causing [adverse effect on teams, reliability, velocity, or cost].

h3. Benefit Hypothesis

We believe [measurable outcome]
will be achieved if [target users/teams]
can successfully [user outcome or capability]
using [this feature].

*Key Metrics:*
* [Metric 1]: current [baseline], target [goal]
* [Metric 2]: current [baseline], target [goal]

h3. Solution Overview

[2-4 sentences: high-level approach. What are we building or changing?
Keep this at the level a skip-level manager could understand.]

*Architectural Impact:*
* Systems affected: [list of services/components]
* New dependencies: [or "None"]
* Operational changes: [e.g., new on-call responsibilities, or "None"]

h3. Scope

*In Scope:*
* [Deliverable 1]
* [Deliverable 2]
* [Deliverable 3]

*Out of Scope:*
* [Item 1] (reason for deferral)
* [Item 2] (reason for deferral)

h3. Acceptance Criteria

* [Observable outcome 1 -- measurable system state, not a task]
* [Observable outcome 2]
* [Observable outcome 3]
* [Observable outcome 4]

h3. Child Epics

* [HYPERFLEET-XXX]: [Epic title] (estimated XX points)
* [HYPERFLEET-XXX]: [Epic title] (estimated XX points)
* [HYPERFLEET-XXX]: [Epic title] (estimated XX points)

h3. Dependencies

* Depends on: [FEATURE/EPIC-XXX or "None"]
* Blocks: [FEATURE/EPIC-XXX or "None"]
* External: [e.g., "Security review", "Quay.io org provisioning", or "None"]

h3. Risks

* [Risk 1] (likelihood: H/M/L, impact: H/M/L): [mitigation]
* [Risk 2] (likelihood: H/M/L, impact: H/M/L): [mitigation]

h3. Open Questions

* [Decision that must be resolved before or during implementation]
* [Question]
EOF
```

### Step 3: Create via CLI

```bash
jira issue create --project ${DUCKY_JIRA_PROJECT:-HYPERFLEET} --type Feature \
  --summary "Feature: Title" \
  --priority High \
  --no-input \
  -b "$(cat /tmp/feature-description.txt)"
```

Notes:
- No `--custom story-points` -- features are planning-level artifacts
- No `--custom activity-type` -- set on child epics/stories instead
- If `--type Feature` returns an error, the JIRA instance may use `--type Initiative` instead

### Discovering Valid Components

Before assigning a component, check what components exist in the project:
```bash
jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND component is not EMPTY" --plain 2>/dev/null | head -20
```
If you know the component, add `--component "ComponentName"` to the create command.

### Step 4: Post-Creation

```bash
jira issue view ${DUCKY_JIRA_PROJECT:-HYPERFLEET}-XXX --plain
```

Manual steps (via web UI):
1. **Link child Epics**: Edit each epic > Link > "is child of" > this Feature
2. **Add Labels**: e.g., quarter label, theme label
3. **Set Target Quarter**: If the JIRA instance supports it

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

### --type Feature Not Recognised
Some JIRA instances use `Initiative` instead of `Feature`. Try:
```bash
--type Initiative
```

### Headers Not Rendering
Ensure space after period: `h3. What` (not `h3.What`)

### --body-file Flag
Does not exist. Use `-b "$(cat /tmp/file.txt)"` instead.

## Prerequisites

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`

## Integration

- **create-epic**: Create child epics after feature is set up
- **ticket-hygiene**: Validate feature quality
