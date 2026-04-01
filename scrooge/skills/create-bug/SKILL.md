---
name: create-bug
description: Creates a JIRA Bug with steps to reproduce, expected/actual behavior, and fix criteria. Activates when users ask to create a bug report, file a bug, or report an issue.
---

# JIRA Bug Creator

## Configuration

- `DUCKY_JIRA_PROJECT`: JIRA project key (default: `HYPERFLEET`)
- `JIRA_BASE_URL`: JIRA instance URL (default: `https://issues.redhat.com`)

## Writing Style

When writing descriptions, apply the ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create a bug", "file a bug", "report a bug"
- "there's a bug in...", "I found a bug"
- "this is broken", "create a bug report"

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

## Priority Guidance

- `Blocker`: Blocks development/testing, must be fixed immediately
- `Critical`: Crashes, data loss, severe memory leak
- `Major`: Major loss of function (default for bugs)
- `Normal`: Minor functional issue
- `Minor`: Cosmetic, easy workaround

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
- What is the bug? (What)
- How do you reproduce it? (Steps)
- What should happen vs what actually happens?
- How severe is it? (Priority)

### Step 2: Create Description File

```bash
cat > /tmp/bug-description.txt << 'EOF'
h3. What

[Description of the bug and its impact. 2-3 sentences.]

h3. Steps to Reproduce

* Step 1
* Step 2
* Step 3

h3. Expected Behavior

[What should happen when following the steps above.]

h3. Actual Behavior

[What actually happens. Include error messages if available.]

h3. Why

* [Who is affected -- users, teams, system reliability]
* [Severity of impact -- data loss, degraded performance, blocking work]

h3. Acceptance Criteria

* Root cause identified and fixed
* Regression test added covering this scenario
* No related side effects introduced
* [Additional criterion specific to this bug]

h3. Technical Notes

* Affected component: {{component-name}}
* Error location: {{file/path}}
* Relevant logs/errors: [describe, add screenshots via web UI]
EOF
```

### Step 3: Create via CLI

```bash
jira issue create --project ${DUCKY_JIRA_PROJECT:-HYPERFLEET} --type Bug \
  --summary "Bug: Brief description (< 100 chars)" \
  --custom story-points=5 \
  --custom activity-type="Quality / Stability / Reliability" \
  --priority Major \
  --no-input \
  -b "$(cat /tmp/bug-description.txt)"
```

**Activity type** defaults to "Quality / Stability / Reliability". Override for:
- Security bugs: "Security & Compliance"
- Customer-reported: "Incidents & Support"

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
2. **Add Labels**: e.g., `bug`, component labels
3. **Add Screenshots/Logs**: Attach files via web UI

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Bug
Summary: [Title]
Points: [X]
Priority: [Priority]
Activity Type: Quality / Stability / Reliability
Link: https://issues.redhat.com/browse/HYPERFLEET-XXX

Manual steps needed:
- Link to parent epic [if applicable]
- Attach screenshots/logs [if available]
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

- **story-pointer**: Estimate complexity of the fix
- **triage**: Validate ticket quality after creation
