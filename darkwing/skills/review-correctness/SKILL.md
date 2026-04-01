---
name: review-correctness
description: Review Round 3 - Detailed line-by-line code review for bugs, security, tests, and performance
allowed-tools: Bash
argument-hint: [PR-URL-or-number]
disable-model-invocation: true
---

# Review Round 3: Correctness

Detailed, bottom-up code review. This is the line-by-line pass where you look for bugs, security issues, test gaps, and performance concerns.

**Cognitive frame:** "Does it work and will we regret this?"

## Arguments
- `$1` (optional): PR URL, number, or omit to auto-detect current branch's PR

## Instructions

### Step 1: Get the diff

```bash
gh pr diff ${1:+$1} 2>/dev/null
```

### Step 2: Review in priority order

Go through the diff and review for issues in this exact priority order:

**Priority 1: Bugs and Correctness**
- Logic errors, off-by-one, nil/null handling
- Race conditions, deadlocks, data races
- Incorrect state transitions
- Wrong assumptions about input data

**Priority 2: Security**
- Input validation gaps
- Auth/authz checks missing or incorrect
- Injection risks (SQL, command, XSS)
- Secrets or credentials in code
- Insecure defaults

**Priority 3: Error Handling**
- Swallowed errors (silently ignoring failures)
- Missing error context (wrapping without info)
- Unhelpful error messages
- Missing cleanup on error paths (resource leaks)

**Priority 4: Test Coverage**
- Are there tests for new/changed behavior?
- Do tests actually verify the behavior (not just "no error")?
- Are edge cases covered (empty input, nil, boundaries)?
- Are tests brittle (coupled to implementation details)?
- If the PR introduces new behavior with no test changes, flag this

**Priority 5: Performance**
- N+1 queries or unnecessary database calls
- Unnecessary allocations or copies
- Missing caching where appropriate
- Operations that won't scale (linear scan where index exists)

### Step 3: Label every comment

**This is mandatory.** Every substantive comment must have exactly one label:

- **Blocker:** Must be fixed before merge. Bugs, security issues, missing critical tests.
- **Warning:** Should be discussed. May or may not need fixing. Performance concerns, questionable error handling.
- **Nit:** Optional improvement. Will not block approval. Style preferences, minor refactoring.

### Step 4: Surface everything in one pass

**Do NOT drip-feed issues.** Mention ALL issues you find in this single pass. The "Death of a Thousand Round Trips" anti-pattern (finding one issue, waiting for a fix, then finding another you could have mentioned the first time) is the primary cause of review loops.

### Do NOT flag

- Style issues that linting would catch (if linters are in CI)
- Issues already raised by other reviewers (from the Gates round)
- Nitpicks on code that is being deleted
- Hypothetical future problems ("what if someone someday...")

## Output Format

```
### Round 3: Correctness

#### [file_path]

**Line [N]** (Blocker):
[Observation]
[Why it matters]
[Suggested fix with code block if applicable]

**Line [N]** (Warning):
[Observation]
[Why it matters]
[Suggested alternative]

**Line [N]** (Nit):
[Brief observation and suggestion]

---

#### Test Coverage Assessment
- New behavior tested: [Yes / No / Partially]
- Edge cases covered: [Yes / No / List gaps]
- Test quality: [Strong / Adequate / Weak / Missing]

---

#### Issue Summary
| Label | Count |
|-------|-------|
| Blocker | X |
| Warning | X |
| Nit | X |
```

## Exit Criteria

All issues have been surfaced and labeled in a single pass. Do not hold anything back for a second round.
