---
name: create-epic
description: Creates a JIRA Epic with scope, success criteria, and child story breakdown. Activates when users ask to create an epic.
---

# JIRA Epic Creator

## Configuration

- `DUCKY_JIRA_PROJECT`: JIRA project key (default: `HYPERFLEET`)
- `JIRA_BASE_URL`: JIRA instance URL (default: `https://redhat.atlassian.net`)

## Writing Style

When writing descriptions, apply the ghostwriter skill for tone.
If `qdrant-find` MCP tool is available, query for "ticket description" style samples.

## When to Use This Skill

- "create an epic", "I need an epic for..."
- "set up an epic for this feature area"
- "create an epic to track..."

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

## Important: Epic-Specific CLI Differences

Epics differ from other types in these critical ways:

1. **Epics require `--custom epic-name="Short Name"`** -- this is a mandatory field
2. **Epics use `--template` instead of `-b`** for the description body
3. **Epics do NOT get story points** -- they aggregate child story points
4. **Activity type is optional** for epics

## Workflow

### Step 1: Gather Requirements

Ask the user if needed:
- What are we building? (What)
- Why does this matter? (Why)
- What's in scope vs out of scope?
- What are the success criteria?
- What are the known risks or dependencies?

### Step 2: Create Description File

```bash
cat > /tmp/epic-description.txt << 'EOF'
h1. Epic Title

h3. What

[What are we building? 2-3 sentences describing the deliverable.]

h3. Why

[Why does this matter? 1-2 sentences on the problem it solves or value it delivers.]

h3. Scope

*In Scope:*
* [Deliverable 1]
* [Deliverable 2]
* [Deliverable 3]

*Out of Scope:*
* [Item 1]
* [Item 2]

h3. Acceptance Criteria

* [Criterion 1 -- observable outcome]
* [Criterion 2]
* [Criterion 3]
* [Criterion 4]
* [Criterion 5]

h3. Dependencies

* Blocked by: [EPIC-XXX or "None"]
* Blocks: [EPIC-XXX or "None"]

h3. Risks

* [Risk 1]: [mitigation strategy]
* [Risk 2]: [mitigation strategy]

h3. Notes

[Additional context, links to design docs, related research.]
EOF
```

### Step 3: Create via CLI

**CRITICAL: Use `--template` for epics, NOT `-b "$(cat ...)"`**

Why: Epic descriptions often contain wiki markup characters (`*`, `{`, `h1.`) that get
mangled by shell expansion when passed through `-b "$(cat ...)"`. The `--template` flag
reads the file directly, bypassing shell interpolation. Other issue types are shorter
and simpler, so `-b "$(cat ...)"` works fine for them.

```bash
jira issue create --project ${DUCKY_JIRA_PROJECT:-HYPERFLEET} --type Epic \
  --summary "Epic: Full Title Here" \
  --custom epic-name="Short Name" \
  --no-input \
  --template /tmp/epic-description.txt
```

Notes:
- `--custom epic-name="Short Name"` is REQUIRED (not `epicName`, not `customfield_12311141`)
- `--template` reads the file directly (unlike `-b "$(cat ...)"` used by other types)
- No `--custom story-points` -- epics aggregate child points
- No `--custom activity-type` -- set on child stories instead

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
1. **Link to Feature**: Edit ticket > Link > "is child of" > Feature (if applicable)
2. **Add Labels**: e.g., component or theme labels
3. **Add Code Examples**: Code blocks don't render via CLI

## Output Format

```
Ticket Created: HYPERFLEET-XXX

Type: Epic
Summary: [Title]
Epic Name: [Short Name]
Link: https://redhat.atlassian.net/browse/HYPERFLEET-XXX

Manual steps needed:
- Link to parent feature [if applicable]
- Add labels [if applicable]
- Create child stories
```

## Troubleshooting

### Epic Name Required Error
```
Error: customfield_12311141: Epic Name is required.
```
Use: `--custom epic-name="Short Name"` (not `epicName` or `customfield_12311141`)

### Headers Not Rendering
Ensure space after period: `h3. What` (not `h3.What`)

### --body-file Flag
Does not exist. For epics use `--template /tmp/file.txt`. For other types use `-b "$(cat /tmp/file.txt)"`.

## Prerequisites

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`

## Integration

- **create-story**: Create child stories after epic is set up
- **ticket-hygiene**: Validate epic quality
