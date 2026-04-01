---
name: create-task
description: Creates a JIRA Task for tech debt, infrastructure, documentation, or spike work. Activates when users ask to create a task, spike, tech debt ticket, or infrastructure work.
---

# JIRA Task Creator

## Configuration

- `DUCKY_JIRA_PROJECT`: JIRA project key (default: `HYPERFLEET`)
- `JIRA_BASE_URL`: JIRA instance URL (default: `https://issues.redhat.com`)

## Writing Style

When writing descriptions, apply the ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create a task", "add a task"
- "create a spike for...", "I need a spike", "investigate..."
- "tech debt ticket for...", "create a tech debt item"
- "infrastructure task", "documentation task"

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

### Step 2: Create Description File

**Default Template (tech debt / infra / docs):**

```bash
cat > /tmp/task-description.txt << 'EOF'
h3. What

[What needs to be done. 2-3 sentences.]

h3. Why

* [Business or technical impact]
* [What problem this solves]

h3. Current State

[What is problematic now -- the pain point or gap.]

h3. Desired State

[What we want instead -- the target outcome.]

h3. Acceptance Criteria

* [Specific, testable criterion 1]
* [Criterion 2]
* [Criterion 3]

h3. Technical Approach

* [How this will be accomplished]
* Files/components affected:
** {{component-1}}
** {{component-2}}
EOF
```

**Spike Template (investigations / research / PoCs):**

```bash
cat > /tmp/task-description.txt << 'EOF'
h3. Research Question

[What we need to answer. Be specific.]

h3. Why

* [Why this investigation matters]
* [What decisions depend on the outcome]

h3. Time Box

[Max time to spend, e.g., "3 days" or "1 sprint"]

h3. Deliverable

[What the output should be: report, recommendation, PoC, ADR, etc.]

h3. Acceptance Criteria

* Research questions answered with evidence
* Deliverable produced and shared with team
* Recommendation documented with trade-offs
* Next steps identified
EOF
```

### Step 3: Create via CLI

```bash
jira issue create --project ${DUCKY_JIRA_PROJECT:-HYPERFLEET} --type Task \
  --summary "Task title (< 100 chars)" \
  --custom story-points=3 \
  --custom activity-type="Future Sustainability" \
  --priority Normal \
  --no-input \
  -b "$(cat /tmp/task-description.txt)"
```

For spikes, prefix the summary: `--summary "[SPIKE] Research question summary"`

**Activity type** defaults to "Future Sustainability". Override when needed:
- Refactoring/quality work: "Quality / Stability / Reliability"
- Security tasks: "Security & Compliance"
- Customer-driven: "Incidents & Support"

Valid activity types: `Associate Wellness & Development`, `Incidents & Support`, `Security & Compliance`, `Quality / Stability / Reliability`, `Future Sustainability`, `Product / Portfolio Work`

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
1. **Link to Epic**: Edit ticket > Link > "is child of" > Epic
2. **Add Labels**: e.g., `tech-debt`, `spike`, `infrastructure`

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Task
Summary: [Title]
Points: [X]
Priority: [Priority]
Activity Type: [Type]
Link: https://issues.redhat.com/browse/HYPERFLEET-XXX

Manual steps needed:
- Link to parent epic [if applicable]
- Add labels [if applicable]
```

## Troubleshooting

### Story Points Not Setting
Use exact syntax: `--custom story-points=X` where X is 0, 1, 3, 5, 8, or 13.

### --body-file Flag
Does not exist. Use `-b "$(cat /tmp/file.txt)"` instead.

## Prerequisites

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`

## Integration

- **story-pointer**: Estimate task complexity
- **triage**: Validate ticket quality after creation
