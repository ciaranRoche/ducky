---
name: review-gates
description: Review Round 1 - Check CI/CD status, automated checks, and existing reviewer comments before diving into code
allowed-tools: Bash
argument-hint: [PR-URL-or-number]
disable-model-invocation: true
---

# Review Round 1: Gates

Check whether the PR passes its automated gates before any human review effort.

**Cognitive frame:** "Does this even build?"

## Arguments
- `$1` (optional): PR URL, number, or omit to auto-detect current branch's PR

## Instructions

### Step 1: Fetch PR metadata

```bash
gh pr view ${1:+$1} --json number,title,state,author,baseRefName,headRefName,additions,deletions,changedFiles,reviewDecision,statusCheckRollup,url 2>/dev/null
```

If no PR found, inform the user and suggest `gh pr list`.

### Step 2: Check CI/CD status

```bash
gh pr checks ${1:+$1} 2>/dev/null
```

Evaluate:
- **Are all CI checks passing?** If not, flag failing checks as the top issue.
- **Are tests passing?** If tests are failing, this is the first thing the author needs to fix.
- **Are linting/static analysis checks passing?** Note these but don't dwell on them.

**If any checks are failing, stop here. That is the review.** Do not proceed to design or correctness rounds. The author needs to fix CI first.

### Step 3: Fetch existing review comments

```bash
# Inline review comments
gh api repos/{owner}/{repo}/pulls/{number}/comments --jq '.[] | "[\(.user.login)] \(.path):\(.line // .original_line) — \(.body)"' 2>/dev/null

# Top-level review comments
gh api repos/{owner}/{repo}/pulls/{number}/reviews --jq '.[] | "[\(.user.login)] \(.state): \(.body)"' 2>/dev/null
```

Build a list of issues already raised by other reviewers. These will be excluded from later rounds to avoid duplicating feedback.

### Step 4: Determine review mode

```bash
gh api user --jq '.login' 2>/dev/null
```

- **Self-review**: If the PR author matches the current user, later rounds should focus on "things I'd catch if I were reviewing someone else's PR"
- **External review**: Full review for another person's PR

## Output Format

```
### Round 1: Gates

#### CI/CD Status
| Check       | Status  |
|-------------|---------|
| [check-name]| PASS / FAIL / PENDING |

[If any checks are failing, call them out and recommend the author fix CI before requesting review]

#### Existing Feedback
- [count] comments from other reviewers
- Key issues already raised: [brief summary]

#### Review Mode
- [Self-review / External review]
- PR Size: +[additions] / -[deletions] across [changedFiles] files

#### Gate Result: PASS / FAIL
[If FAIL: "CI is failing. Fix automated checks before proceeding with code review."]
[If PASS: "All gates clear. Proceed to design review."]
```

## Exit Criteria

- **PASS**: All automated checks pass. Proceed to Round 2 (Design).
- **FAIL**: One or more checks are failing. Stop here. The failing checks ARE the review.
