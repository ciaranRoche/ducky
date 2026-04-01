---
name: review-design
description: Review Round 2 - Top-down architecture and design assessment of a PR before detailed code review
allowed-tools: Bash
argument-hint: [PR-URL-or-number]
disable-model-invocation: true
---

# Review Round 2: Design

Assess the PR's architectural approach before diving into line-by-line review. This prevents Priority Inversion (wasting time on details when the overall approach needs rethinking).

**Cognitive frame:** "Is the approach right?"

## Arguments
- `$1` (optional): PR URL, number, or omit to auto-detect current branch's PR

## Instructions

### Step 1: Read the PR as a whole

```bash
# PR description and metadata
gh pr view ${1:+$1} --json number,title,body,additions,deletions,changedFiles,baseRefName,headRefName 2>/dev/null

# File list (what areas of the codebase are touched?)
gh pr diff ${1:+$1} --stat 2>/dev/null

# Full diff for understanding
gh pr diff ${1:+$1} 2>/dev/null
```

### Step 2: Assess architectural fit

Review the change at the system level. Ask yourself:

- **Does this change belong here?** Is it in the right layer/module/package?
- **Does it follow existing patterns?** Or does it introduce a new pattern where one already exists?
- **Is the abstraction level right?** Too abstract (over-engineered for the use case)? Too concrete (should be generalized)?
- **Are responsibilities correctly separated?** Single responsibility at the component level?
- **Are there API surface changes?** If so, are they consistent with existing API design?

### Step 3: Evaluate scope

- **Is this PR doing too much?** Should it be split into smaller, independently reviewable changes?
- **Is the scope clear from the description?** Does the PR description explain what and why?
- **Are there unrelated changes mixed in?** Formatting changes, refactoring, or feature work bundled together?

If the PR exceeds 400 LOC of meaningful changes, note this and recommend splitting unless the changes are tightly coupled.

### Step 4: Check for design-level concerns

Look for issues that would require a fundamental restructure, not just a line-level fix:

- Wrong data model or schema design
- Missing or incorrect error propagation strategy
- Tight coupling that will create problems at scale
- Security architecture issues (auth model, trust boundaries)
- Missing or wrong concurrency model
- Changes that will be hard to roll back

**If a fundamental design issue exists, raise it as a Blocker and recommend NOT proceeding to detailed review.** There is no point reviewing individual lines of code that will need to be rewritten.

## Output Format

```
### Round 2: Design

#### Approach Assessment
[2-3 sentences on what this PR does and whether the approach makes sense]

#### Architectural Fit
- [Does it follow existing patterns? Y/N with explanation]
- [Is the abstraction level right? Y/N with explanation]
- [Responsibilities correctly separated? Y/N with explanation]

#### Scope Assessment
- PR size: [appropriate / too large / consider splitting]
- Unrelated changes: [none / list of unrelated changes]

#### Design Concerns
[List any design-level issues as Blocker/Warning]

**Blocker: [issue]**
[Observation, Why It Matters, Suggested Alternative]

**Warning: [issue]**
[Observation, Why It Matters, Suggested Alternative]

#### Design Result: PASS / BLOCKER
[If BLOCKER: "Fundamental design concern raised. Resolve before detailed review."]
[If PASS: "Design is sound. Proceed to correctness review."]
```

## Exit Criteria

- **PASS**: Architecture is sound. Proceed to Round 3 (Correctness).
- **BLOCKER**: Fundamental design issue found. Raise it and stop. Do not proceed to line-by-line review. This prevents Priority Inversion.
