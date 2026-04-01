---
name: pr-status
description: Check status of a PR - CI, reviews, merge readiness
allowed-tools: Bash
argument-hint: [PR-URL-or-number]
---

# PR Status

Check the full status of a pull request: CI checks, review status, and merge readiness.

## Arguments
- `$1` (optional): PR URL or number. If omitted, detects the PR for the current branch.

## Instructions

### Step 1: Fetch PR details

```bash
gh pr view ${1:+$1} --json number,title,state,author,baseRefName,headRefName,additions,deletions,changedFiles,mergeable,reviewDecision,statusCheckRollup,reviews,url 2>/dev/null
```

### Step 2: Fetch CI checks

```bash
gh pr checks ${1:+$1} 2>/dev/null
```

### Step 3: Fetch review details

```bash
gh pr view ${1:+$1} --json reviews --jq '.reviews[] | "\(.author.login): \(.state)"' 2>/dev/null
```

## Output Format

```
### PR #[number]: [title]

**URL**: [url]
**Author**: @[author]
**Branch**: [head] → [base]
**State**: [Open/Closed/Merged]

---

| Check       | Status                    |
|-------------|---------------------------|
| CI          | PASS / FAIL / PENDING     |
| Reviews     | X/Y approved              |
| Mergeable   | Yes / No / Conflicting    |

---

#### CI Checks
| Name                | Status  | Duration |
|---------------------|---------|----------|
| [check-name]        | PASS    | 3m 12s   |
| [check-name]        | FAIL    | 1m 45s   |

#### Review Status
| Reviewer      | Decision          |
|---------------|-------------------|
| @[reviewer]   | APPROVED          |
| @[reviewer]   | CHANGES_REQUESTED |
| @[reviewer]   | PENDING           |

---

#### Action Items
1. [What needs to happen before this can merge]
2. [Any failing checks to investigate]
3. [Any reviews still pending]
```

## Notes
- If no PR exists for the current branch, inform the user
- Highlight any failing CI checks prominently
- Call out stale reviews (approved but with new commits since)
