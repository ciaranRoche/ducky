---
name: jira-create-story
description: Creates a JIRA Story with user story format, acceptance criteria, and technical notes. Activates when users ask to create a story or user story.
---

# JIRA Story Creator

## Configuration

- `DUCKY_JIRA_PROJECT`: JIRA project key (default: `HYPERFLEET`)
- `JIRA_BASE_URL`: JIRA instance URL (default: `https://issues.redhat.com`)

## Writing Style

When writing descriptions, apply the ducky-ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create a story", "write a story", "add a user story"
- "I need a story for..."
- "create a story for this feature"

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
- 0: Tracking only (no work)
- 1: Trivial (< half day)
- 3: Straightforward (1-2 days)
- 5: Medium complexity (2-4 days)
- 8: Large, may need design doc (1+ week)
- 13: Too large -- break it down

For detailed estimation, reference the **jira-story-pointer** skill.

## Workflow

### Step 1: Gather Requirements

Ask the user if needed:
- What needs to be done? (What)
- Who benefits and why? (Why)
- How will we know it's done? (Acceptance Criteria)
- How complex is this? (Story Points)

### Step 2: Create Description File

```bash
cat > /tmp/story-description.txt << 'EOF'
h3. What

[User story or technical description. 2-4 sentences.]

h3. Why

* [Reason 1 -- who benefits and how]
* [Reason 2 -- what problem it solves]

h3. Acceptance Criteria

* [Given/When/Then or testable criterion 1]
* [Criterion 2]
* [Criterion 3]

h3. Technical Notes

* Implementation approach: [brief description]
* Files/components affected:
** {{component-1}}
** {{component-2}}
* API/DB changes: [if any, or "None"]

h3. Out of Scope

* [Explicit exclusion 1]
EOF
```

### Step 3: Create via CLI

```bash
jira issue create --project ${DUCKY_JIRA_PROJECT:-HYPERFLEET} --type Story \
  --summary "Story title (< 100 chars)" \
  --custom story-points=5 \
  --custom activity-type="Product / Portfolio Work" \
  --priority Normal \
  --no-input \
  -b "$(cat /tmp/story-description.txt)"
```

**Activity type** defaults to "Product / Portfolio Work". Override when needed:
- Infrastructure stories: "Future Sustainability"
- Reliability/quality stories: "Quality / Stability / Reliability"
- Security work: "Security & Compliance"

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
2. **Add Labels**: If applicable
3. **Add Code Examples**: Code blocks don't render via CLI

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Story
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

- **jira-story-pointer**: Detailed estimation methodology
- **jira-triage**: Validate ticket quality after creation
