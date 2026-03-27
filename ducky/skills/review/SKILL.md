---
name: review
description: Review a PR using gh CLI, with feedback in your personal writing style
allowed-tools: Bash, mcp__writing-samples__qdrant-find
argument-hint: [PR-URL-or-number]
disable-model-invocation: true
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

### Step 2: Fetch the diff and existing review comments

```bash
# Get the diff
gh pr diff $1 2>/dev/null

# Get existing review comments so we don't repeat feedback already given
gh api repos/{owner}/{repo}/pulls/{number}/comments --jq '.[] | "[\(.user.login)] \(.path):\(.line // .original_line) — \(.body)"' 2>/dev/null

# Get top-level review comments
gh api repos/{owner}/{repo}/pulls/{number}/reviews --jq '.[] | "[\(.user.login)] \(.state): \(.body)"' 2>/dev/null
```

### Step 3: Check CI/CD and test status

This is the **highest priority** part of the review. Before looking at code quality, verify the PR's automated checks.

```bash
# Fetch all check statuses
gh pr checks $1 2>/dev/null
```

Evaluate:
- **Are all CI checks passing?** If not, flag failing checks as the top issue. No point reviewing code that doesn't build or pass CI.
- **Are tests passing?** If tests are failing, this is the first thing the author needs to fix.
- **Are linting/static analysis checks passing?** Don't manually flag style issues that a linter would catch.

### Step 4: Evaluate test coverage of the changes

Look at the diff and determine:
- **Are there tests for the new/changed behavior?** If new functionality was added or existing behavior changed, there should be corresponding test changes.
- **Are edge cases covered?** Look for boundary conditions, error paths, and nil/empty inputs.
- **Are tests testing the right thing?** Tests that only assert "no error" without checking the result are weak. Tests that are tightly coupled to implementation details are brittle.

If the PR has no test changes and introduces new behavior, this should be flagged as the primary concern.

### Step 5: Filter out already-given feedback

Review the existing comments fetched in Step 2. **Do not raise issues that another reviewer has already called out.** If someone has already flagged a bug, missing test, or style issue, skip it. Only add new value.

If you agree with existing feedback that hasn't been addressed, you can briefly reinforce it ("agree with @reviewer here") but do not restate the full concern.

### Step 6: Determine review mode

- **Self-review**: If the PR author matches `gh api user --jq '.login'`, focus on what you might have missed before submitting
- **External review**: Full review with draft comments for another person's PR

### Step 7: Query writing style

If the `qdrant-find` MCP tool is available, run 2-3 queries:
1. Topic query matching the PR content (e.g., "helm chart security", "database migration")
2. Comment type query (e.g., "nitpick with code suggestion", "architecture concern")
3. Tone query (e.g., "constructive code review", "positive praise with feedback")

Use retrieved samples to calibrate voice and tone.

### Step 8: Analyze the diff

With CI/tests/existing feedback already handled, now review the code for:
- **Bugs**: Logic errors, off-by-one, nil/null handling, race conditions
- **Security**: Input validation, auth checks, injection risks, secrets exposure
- **Performance**: N+1 queries, unnecessary allocations, missing caching
- **Error handling**: Swallowed errors, missing context, unhelpful messages
- **Consistency**: Does the code follow existing patterns in the codebase?

Do **not** flag:
- Style issues that linting would catch (if linters are in CI)
- Issues already raised by other reviewers
- Nitpicks on code that is being deleted

### Step 9: Write review comments

Apply the ghostwriter skill for all written output:
- Use the three-part structure: **Observation** → **Why It Matters** → **Suggested Fix**
- Apply severity-to-tone mapping from ghostwriter
- Write a summary comment with severity calibration

## Output Format

```
### PR Review: #[number] - [title]

**Author**: @[author]
**Branch**: [head] → [base]
**Changes**: +[additions] / -[deletions] across [changedFiles] files

---

#### CI/CD Status
| Check       | Status                    |
|-------------|---------------------------|
| [check-name]| PASS / FAIL / PENDING     |

[If any checks are failing, call them out here with what needs to happen]

#### Test Coverage
[Assessment of whether the changes have adequate test coverage. Flag gaps.]

---

**Summary**: [Ghostwriter-style severity calibration]

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

## Review Priority Order

1. **CI/CD status** — if checks are failing, that's the review
2. **Test coverage** — are the changes tested?
3. **Bugs and correctness** — logic errors, security issues
4. **Design and patterns** — consistency, performance
5. **Nits** — only if everything above is clean

## Notes
- For self-reviews, focus on "things I'd catch if I were reviewing someone else's PR"
- For external reviews, be thorough but fair
- Always use ghostwriter tone: casual, direct, empathetic, teaching-oriented
- Never use dashes as connectors, no corporate language, no emoji
- Every substantive comment must explain "why", not just "what"
- Never duplicate feedback already given by another reviewer
