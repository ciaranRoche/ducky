---
name: pr-create
description: Create a PR with a well-structured description in your writing style
allowed-tools: Bash, mcp__writing-samples__qdrant-find
argument-hint: [base-branch]
disable-model-invocation: true
---

# Create PR

Create a pull request with a well-structured description written in your personal style.

## Arguments
- `$1` (optional): Base branch to merge into (default: auto-detected by gh)

## Instructions

### Step 1: Gather context

```bash
# Current branch
git branch --show-current

# Resolve base branch once
BASE=${1:-$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')}

# Changes vs base
git log --oneline $(git merge-base HEAD $BASE)..HEAD

# Diff stats
git diff --stat $BASE...HEAD

# Full diff for understanding
git diff $BASE...HEAD
```

### Step 2: Check for existing PR

```bash
# Check if a PR already exists for this branch
gh pr view --json number,title,url 2>/dev/null
```

If a PR already exists, show it to the user and ask whether they want to:
- Update the existing PR's description
- Create a new PR to a different base branch

If no PR exists, continue to Step 3.

### Step 3: Query writing style

If the `qdrant-find` MCP tool is available, query for:
1. "pull request description" style samples
2. "technical summary" style samples

Use retrieved samples to match the user's voice for PR descriptions.

### Step 4: Generate PR title and body

**Title rules:**
- Under 72 characters
- Imperative mood ("Add feature" not "Added feature")
- Clear and specific

**Body structure:**
- Summary: 2-3 sentences explaining what and why
- Changes: bullet list of key changes
- Testing: how this was tested
- Use ghostwriter tone throughout

### Step 5: Create the PR

```bash
gh pr create \
  --title "[generated title]" \
  --body "[generated body]" \
  ${1:+--base $1}
```

### Step 6: Return results

## Output Format

```
### PR Created: #[number]

**Title**: [title]
**URL**: [url]
**Branch**: [head] → [base]

---

[PR body as written]
```

## Notes
- If there are no commits ahead of base, inform the user
- If a PR already exists for this branch, show it instead of creating a duplicate
- Use ghostwriter style: casual, precise, no dashes as connectors
