---
name: review
description: Review a PR using a structured multi-pass approach with feedback in your personal writing style
allowed-tools: Bash, mcp__writing-samples__qdrant-find
argument-hint: [PR-URL-or-number]
disable-model-invocation: true
---

# PR Review

Review a pull request using a structured multi-pass approach. Each round has a focused cognitive frame and clear exit criteria to prevent review loops.

## Arguments
- `$1` (optional): PR URL, number, or omit to auto-detect current branch's PR

## Review Pipeline

This skill orchestrates 4 review rounds in sequence. Each round can also be invoked independently.

### Round 0: Triage (inline)

Before reading any code, classify the PR to determine review depth:

```bash
gh pr view ${1:+$1} --json number,title,additions,deletions,changedFiles,body 2>/dev/null
```

| PR Type | Depth | Strategy |
|---------|-------|----------|
| Hotfix / site-down | Quick sanity | Skip to Correctness, fast approval |
| Config / docs changes | Lightweight | Gates + quick Correctness, skip Design |
| Small feature (< 200 LOC) | Standard | All rounds, single sitting |
| Medium feature (200-400 LOC) | Full multi-pass | All rounds |
| Large feature (400+ LOC) | Architecture-focused | Warn user, focus on Design, limit line-by-line |
| Refactoring (no behavior change) | Test-focused | Gates + Correctness (trust test suite) |

### Round 1: Gates

Run the **review-gates** skill:
- Check CI/CD status, test results, linting
- Fetch existing reviewer comments (for dedup in later rounds)
- Determine self-review vs external review mode

**If CI is failing, stop here. That is the review.**

### Round 2: Design

Run the **review-design** skill:
- Assess architectural fit, abstraction level, scope
- Check for design-level concerns that would require restructuring

**If a fundamental design issue exists, raise it as a Blocker and skip Round 3.** Do not waste effort reviewing line-by-line code that will be rewritten. This prevents Priority Inversion.

### Round 3: Correctness

Run the **review-correctness** skill:
- Line-by-line review for bugs, security, error handling, tests, performance
- Label every comment: `Blocker:`, `Warning:`, or `Nit:`
- Surface ALL issues in one pass (no drip-feeding)

### Round 4: Verdict

Run the **review-verdict** skill:
- Synthesize findings from all rounds
- Apply ghostwriter style to all output
- Determine verdict: APPROVE / REQUEST CHANGES / COMMENT
- Apply termination criteria to prevent loops

## Anti-Patterns This Pipeline Prevents

| Anti-Pattern | How It's Prevented |
|---|---|
| **Priority Inversion** | Design review (Round 2) happens before detailed review (Round 3). Design blockers skip the detail round entirely. |
| **Death of a Thousand Round Trips** | Round 3 requires surfacing ALL issues in one pass. No drip-feeding. |
| **No Termination Criteria** | Round 4 has explicit rules: approve when code improves the system, "LGTM nits aside" is valid, escalate after 2 round-trips. |
| **Style Wars** | Do not flag style issues that linters catch. Ever. |
| **Rubber-stamping** | Each round has a specific cognitive frame and checklist. |

## Notes
- For self-reviews, focus on "things I'd catch if I were reviewing someone else's PR"
- For external reviews, be thorough but fair
- Always use ghostwriter tone: casual, direct, empathetic, teaching-oriented
- Never use dashes as connectors, no corporate language, no emoji
- Every substantive comment must explain "why", not just "what"
- Never duplicate feedback already given by another reviewer
- If a PR is > 400 LOC, recommend splitting before doing a full review
