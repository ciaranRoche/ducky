---
description: Review a PR using gh CLI, with feedback in your personal writing style
allowed-tools: Bash
argument-hint: [PR-URL-or-number]
---

# PR Review

Review a pull request and provide feedback in your personal writing style.

## Arguments
- `$1` (optional): PR URL, number, or omit to auto-detect current branch's PR

## Instructions

### Step 1: Identify the PR

```bash
# If argument provided, use it directly
# If no argument, detect PR for current branch
gh pr view ${1:---json number,title,url} --json number,title,url,body,additions,deletions,changedFiles,author,baseRefName,headRefName,reviewDecision,statusCheckRollup 2>/dev/null
```

If no PR found, inform the user and suggest `gh pr list`.

### Step 2: Fetch the diff

```bash
gh pr diff $1 2>/dev/null
```

### Step 3: Determine review mode

- **Self-review**: If the PR author matches `gh api user --jq '.login'`, focus on what you might have missed before submitting
- **External review**: Full review with draft comments for another person's PR

### Step 4: Query writing style

If the `qdrant-find` MCP tool is available, run 2-3 queries:
1. Topic query matching the PR content (e.g., "helm chart security", "database migration")
2. Comment type query (e.g., "nitpick with code suggestion", "architecture concern")
3. Tone query (e.g., "constructive code review", "positive praise with feedback")

Use retrieved samples to calibrate voice and tone.

### Step 5: Analyze the diff

Review for:
- **Bugs**: Logic errors, off-by-one, nil/null handling, race conditions
- **Security**: Input validation, auth checks, injection risks, secrets exposure
- **Performance**: N+1 queries, unnecessary allocations, missing caching
- **Style**: Naming, consistency with existing patterns, readability
- **Testing**: Missing test cases, edge cases not covered, test quality
- **Documentation**: Missing or outdated comments, README updates needed
- **Error handling**: Swallowed errors, missing context, unhelpful messages

### Step 6: Write review comments

Apply the ghostwriter skill for all written output:
- Use the three-part structure: **Observation** → **Why It Matters** → **Suggested Fix**
- Apply severity-to-tone mapping from ghostwriter
- Write a summary comment with severity calibration

### Step 7: Present the review

## Output Format

```
### PR Review: #[number] - [title]

**Author**: @[author]
**Branch**: [head] → [base]
**Changes**: +[additions] / -[deletions] across [changedFiles] files

---

**Summary**: [Ghostwriter-style severity calibration, e.g., "Left a few comments, nothing blocking, mostly just stuff to tighten up before merging."]

---

#### [file_path]

**Line [N]** ([severity: Bug/Nit/Suggestion/Question/Blocker]):
[Ghostwriter-style comment with observation, why it matters, suggested fix]

---

### Verdict: [APPROVE / REQUEST CHANGES / COMMENT]

**Ready to submit this review?**
- Yes: Will post via `gh pr review [number] --comment --body "..."`
- No: Review stays as a draft for you to refine
```

## Notes
- This command runs inline. For autonomous PR review, use the `reviewer` agent instead.
- For self-reviews, focus on "things I'd catch if I were reviewing someone else's PR"
- For external reviews, be thorough but fair
- Always use ghostwriter tone: casual, direct, empathetic, teaching-oriented
- Never use dashes as connectors, no corporate language, no emoji
- Every substantive comment must explain "why", not just "what"
