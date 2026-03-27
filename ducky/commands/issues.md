---
description: GitHub issue management - create, search, comment, close, view
allowed-tools: Bash, mcp__writing-samples__qdrant-find
argument-hint: [action] [args]
disable-model-invocation: true
---

# GitHub Issues

Manage GitHub issues: create, search, list, comment, close, view.

## Arguments
- `$1`: Action — `create`, `search`, `list`, `comment`, `close`, `view`
- `$2+`: Action-specific arguments

## Instructions

### Action: list (default if no action given)

```bash
gh issue list --limit 20
```

Present results grouped by labels or milestones.

### Action: view [number]

```bash
gh issue view $2
```

### Action: create

Interactive issue creation:

1. Ask for title, or infer from context
2. Ask for description, or draft one from context
3. Query `qdrant-find` MCP tool for "issue description" style if available
4. Write the body in ghostwriter style

```bash
gh issue create --title "[title]" --body "[body]"
```

### Action: search [query]

```bash
gh issue list --search "$2" --limit 20
```

### Action: comment [number] [message]

1. If message provided, use it; otherwise ask what to comment
2. Apply ghostwriter style to the comment

```bash
gh issue comment $2 --body "[ghostwriter-styled message]"
```

### Action: close [number]

```bash
gh issue close $2
```

Confirm before closing. Show the issue title first.

## Output Format

### For list/search:

```
### Open Issues

| # | Title | Labels | Assignee | Updated |
|---|-------|--------|----------|---------|
| 42 | [title] | bug, api | @user | 2d ago |
```

### For create:

```
### Issue Created: #[number]
**Title**: [title]
**URL**: [url]
```

### For view:

```
### Issue #[number]: [title]
**State**: Open/Closed
**Author**: @[author]
**Labels**: [labels]
**Assignees**: [assignees]

---

[body]

---

### Comments ([count])
[Recent comments]
```

## Notes
- Use ghostwriter style for any written content (issue bodies, comments)
- When creating issues, suggest relevant labels if the repo has them
- For search, use GitHub's search syntax (e.g., `is:open label:bug`)
