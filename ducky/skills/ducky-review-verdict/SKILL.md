---
name: ducky-review-verdict
description: Review Round 4 - Synthesize findings from all review rounds into a verdict with ducky-ghostwriter style
allowed-tools: Bash, mcp__writing-samples__qdrant-find
argument-hint: [PR-URL-or-number]
disable-model-invocation: true
---

# Review Round 4: Verdict

Synthesize all findings from the previous rounds into a final review verdict. Apply ducky-ghostwriter style to all written output.

**Cognitive frame:** "What's the call?"

## Arguments
- `$1` (optional): PR URL, number, or omit to auto-detect current branch's PR

## Instructions

### Step 1: Query writing style

If the `qdrant-find` MCP tool is available, run 2-3 queries:
1. Topic query matching the PR content (e.g., "helm chart security", "database migration")
2. Comment type query (e.g., "code review summary", "constructive feedback")
3. Tone query (e.g., "review verdict", "approval with comments")

Use retrieved samples to calibrate voice and tone.

### Step 2: Determine the verdict

Based on the findings from Gates, Design, and Correctness rounds, apply these termination criteria:

**APPROVE when:**
- Zero Blockers remain
- All Warnings have been acknowledged (fixed or explicitly deferred)
- CI passes
- The code is better than what it replaces

**APPROVE with comments ("LGTM, nits aside") when:**
- Zero Blockers remain
- Only Warnings and Nits remain
- The remaining issues are genuinely non-blocking
- Trust the author to handle minor fixes without another review cycle

**REQUEST CHANGES when:**
- One or more Blockers exist
- Design-level concerns were raised in Round 2
- CI is failing (from Round 1)

**COMMENT when:**
- You have questions that need answers before you can decide
- The PR needs clarification on intent or scope

### Step 3: Apply the loop-breaking rule

If this is the 2nd or later review cycle on this PR (check review history from Round 1), and remaining issues are all Warnings or Nits:
- Recommend approving with comments
- Suggest a synchronous conversation (call, pairing session) instead of another async round-trip
- The goal is forward progress, not perfection. Google's rule: "approve when the CL definitely improves the overall code health of the system, even if it isn't perfect"

### Step 4: Write the review

Apply the ducky-ghostwriter skill for all written output:
- Use the three-part structure: **Observation** → **Why It Matters** → **Suggested Fix**
- Apply severity-to-tone mapping from ducky-ghostwriter
- Write a summary comment with severity calibration

## Output Format

```
### PR Review: #[number] - [title]

**Author**: @[author]
**Branch**: [head] → [base]
**Changes**: +[additions] / -[deletions] across [changedFiles] files

---

#### CI/CD Status
| Check       | Status  |
|-------------|---------|
| [check-name]| PASS / FAIL / PENDING |

#### Design Assessment
[1-2 sentences on architectural approach]

#### Test Coverage
[Assessment of whether the changes have adequate test coverage]

---

**Summary**: [Ghostwriter-style severity calibration, e.g., "Left a few comments, nothing blocking, mostly just stuff to tighten up before merging."]

---

#### Findings

**[file_path] Line [N]** (Blocker/Warning/Nit):
[Ghostwriter-style comment with observation, why it matters, suggested fix]

---

### Verdict: [APPROVE / REQUEST CHANGES / COMMENT]

[If APPROVE: brief positive note on what was done well]
[If REQUEST CHANGES: summary of what needs to happen]
[If COMMENT: list of questions that need answers]

**Ready to submit this review?**
- Yes: Will post via `gh pr review [number] --[approve|request-changes|comment] --body "..."`
- No: Review stays as a draft for you to refine
```

## Termination Criteria

Reviews should converge, not loop. Apply these rules:

1. **Never block on Nits.** If remaining issues are all Nits, approve.
2. **"LGTM, nits aside" is a valid and healthy outcome.** Use it.
3. **After 2 round-trips, escalate.** Recommend synchronous conversation instead of another comment thread.
4. **Approve when the code improves the system**, even if it isn't perfect.
5. **Drip-feeding issues across multiple rounds is an anti-pattern.** If you find issues, surface them all at once.

## Notes
- Always use ducky-ghostwriter tone: casual, direct, empathetic, teaching-oriented
- Never use dashes as connectors, no corporate language, no emoji
- Every substantive comment must explain "why", not just "what"
- Never duplicate feedback already given by another reviewer
