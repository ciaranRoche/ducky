---
name: ticket-triage
description: Interactive triage session - collaboratively assess a ticket's validity,
  completeness, and sprint-readiness. Activates when users ask to "triage a ticket",
  "is this ticket ready?", or "walk me through this ticket".
allowed-tools:
  - mcp__atlassian__jira_get_issue
  - mcp__atlassian__jira_search
  - mcp__atlassian__jira_update_issue
  - mcp__atlassian__jira_get_project_components
  - mcp__atlassian__jira_get_project_versions
  - mcp__atlassian__jira_search_fields
  - mcp__atlassian__jira_get_issue_development_info
  - Bash
  - Read
  - Grep
  - Glob
argument-hint: <ticket-key>
---

# JIRA Ticket Triage

Interactive triage session. Walk through a ticket together, assess whether it's actually ready for sprint, and fix issues along the way. Uses `mcp__atlassian__*` MCP tools for JIRA data and `Bash`/`Read`/`Grep`/`Glob` for codebase and GitHub validation.

This is not a field checklist -- that's `/hygiene`. This is a conversation about whether the ticket makes sense, has clear scope, and is ready to be picked up.

## Arguments
- `$1`: Ticket key (e.g., HYPERFLEET-123). If omitted, ask the user.

## Custom Field Reference

| Field ID | Name | Notes |
|----------|------|-------|
| `customfield_10016` | Story point estimate | Next-gen — check first |
| `customfield_10028` | Story Points | Classic fallback |
| `customfield_10464` | Activity Type | Select dropdown |

**Important:** These custom fields are NOT returned by default by MCP tools. When calling `mcp__atlassian__jira_get_issue` or `mcp__atlassian__jira_search`, you **must** include them in the `fields` parameter:
```
fields: "summary,description,issuetype,status,priority,labels,assignee,reporter,created,updated,components,fixVersions,customfield_10016,customfield_10028,customfield_10464"
```

## When to Use This Skill

- "triage TICKET-KEY", "triage this ticket"
- "is this ticket ready for sprint?"
- "walk me through this ticket"
- "can you review this ticket with me?"
- "let's look at TICKET-KEY together"

## Interaction Style

Follow the conversational patterns from the pair-programmer skill:
- **Short responses**: 1-3 sentences per turn to keep momentum
- **One question at a time**: Don't dump all assessments at once
- **User-driven pacing**: If they say "looks good" for an area, move on
- **Casual tone**: Use ghostwriter skill for voice
- **Fix as you go**: Offer to fix issues when found, confirm before changing anything
- **No lecturing**: Ask questions, surface concerns, let the user decide

## Triage Flow

### Step 1: Fetch and Ground

Use `mcp__atlassian__jira_get_issue` with the ticket key to pull full details.

Present a brief summary:
- Title, type, status, assignee
- Story points (if set), component, activity type
- Fix Version (if set)

Then ask a grounding question to start the conversation:
- "What's your take on this one -- does the scope feel right?"
- "How confident are you this is sprint-ready?"
- "Any context on this one that isn't in the description?"

Pick the question that fits -- if the ticket looks thin, lean toward scope. If it looks complete, lean toward confidence.

### Step 2: Quick Hygiene Check

Run the 6 required field checks inline (from the data already fetched):

1. Title: clear, actionable, under 100 characters
2. Description: detailed (> 100 characters)
3. Acceptance criteria: at least 2 testable criteria
4. Story points: set (scale: 0, 1, 3, 5, 8, 13) — check `customfield_10016` then `customfield_10028`
5. Component: set and valid — validate against `mcp__atlassian__jira_get_project_components`
6. Activity type: set for capacity planning — check `customfield_10464`

Report briefly: "Quick field check: 5/6 look good. Missing Activity Type. Want me to set that now, or dig into the content first?"

Also note Fix Version status (informational, not counted in the 6-field score):
- If set: "Fix Version: [version]"
- If not set: "Fix Version: not set"

If everything passes: "Fields look clean -- let me check a few things against the codebase."

If there are mechanical gaps, offer to fix them immediately:
- Missing story points: "Want me to set this? What feels right -- 3? 5?"
- Missing activity type: "This looks like [type] to me -- sound right?"
- Missing component: Check valid components first, then suggest

### Step 3: Validate

Check whether the ticket's references to code, PRs, and external resources are still accurate and relevant. This catches stale tickets where the problem has been fixed, files have moved, or linked work has already landed.

#### Detect Validatable Content

Scan the description (already fetched in Step 1) for signals worth checking:
- File paths, function names, error messages, code snippets
- GitHub URLs (PRs, issues, commits)
- JIRA ticket keys mentioned in the body text
- API endpoint references

Also check the JIRA development panel using `mcp__atlassian__jira_get_issue_development_info` with the ticket key.

**If no signals are found and no development info exists:** Skip gracefully. "Nothing in the description references specific code, PRs, or external links. Moving on to content."

#### Development Activity

If `mcp__atlassian__jira_get_issue_development_info` returns linked PRs, branches, or commits:

- **Merged PRs**: "There's a merged PR linked (#42). Has this work already been completed?"
- **Active branches**: "I see a branch linked. Is someone already working on this?"
- **Declined PRs**: "A linked PR was declined. Does the approach need rethinking?"

If there's nothing linked, don't mention it. Move on.

#### Code Verification

If the description references files, functions, error messages, or API endpoints, verify them against the local codebase:

1. Use `Glob` to check referenced file paths exist
2. Use `Grep` to search for function names, error messages, and API route patterns
3. Use `Read` to examine specific files and lines for context

Adapt checks to the ticket type:
- **Bugs**: Focus on whether the bug still exists. Search for the error, check if the function/logic has changed.
- **Stories**: Focus on whether the code area described matches current state. Check API endpoints, file structure.
- **Tasks**: Focus on whether referenced infrastructure, config, or docs still exist and are as described.

Surface findings as questions:
- "The description mentions `handleWidgetCrash()` in `src/widgets/handler.go`, but that function seems to have been renamed to `processWidgetError()`. Does the ticket need updating?"
- "The file referenced in the description doesn't exist anymore. Was this addressed in another ticket?"
- "The error message still matches what's in the code at line 142. Bug looks like it's still present."

If everything checks out: "Code references look accurate."

#### Link Verification

If the description contains GitHub URLs or JIRA ticket references:

**GitHub links:**
```bash
gh pr view <URL> --json state,title,mergedAt 2>/dev/null
gh issue view <URL> --json state,title 2>/dev/null
```

**JIRA references in body text** (ticket keys like PROJ-123 mentioned in description, not formal issue links):
- Fetch with `mcp__atlassian__jira_get_issue` using `fields: "summary,status"`
- Check if they're Done/Closed (dependency may be resolved) or in an unexpected state

Surface findings as questions:
- "PR #87 linked in the description was merged 3 months ago. Still relevant, or should the description be updated?"
- "HYPERFLEET-456 mentioned as a dependency is already Done. Can we remove that note?"
- "GitHub issue #34 is still open. That dependency is still active."

If all links check out: "Links and references look good."

#### Transition

After completing checks (or skipping):
- If issues were found: "Found a few things worth updating. Want to fix those now, or talk through the content first?"
- If everything is clean: "Everything checks out. Let's look at the content."

If the user wants to fix issues, apply description updates using `mcp__atlassian__jira_update_issue` before moving to Step 4.

#### Scoping Constraints

- Check at most 5 file paths, 5 function references, 5 links. If the description has more, spot-check the key ones.
- Only check the current working directory's repo. If the ticket references a different repo, note it and move on.
- Never run code or tests. Read and search only.
- Never modify source code files. Only update the JIRA ticket description if the user confirms.
- One finding at a time — consistent with the interaction style.

### Step 4: Content Assessment

Work through these areas one at a time. Ask one question, wait for the user's response, then move to the next area. Skip areas the user is confident about.

#### Scope Clarity
- "Is the description clear enough that someone unfamiliar with the codebase could pick this up?"
- "Could you explain what 'done' looks like for this in one sentence?"
- "Are there implicit assumptions here that should be made explicit?"

If the description is thin or vague, suggest improvements. Offer to update the description.

#### Acceptance Criteria Quality
- "Are these criteria actually testable? Could QA write test cases from them?"
- "Are there edge cases or error scenarios not covered?"
- "Would you pass a PR that only satisfied these criteria, or would you expect more?"

If criteria are weak, suggest better ones and offer to update.

#### Dependencies and Risk
- "Does this block or get blocked by anything?"
- "Does this touch shared services or APIs that other teams own?"
- "What could go wrong during implementation that isn't accounted for here?"

If dependencies exist, offer to add links.

#### Estimation Sanity
- "Does the story point estimate feel right given what we've discussed?"
- "Is this actually one sprint's worth of work, or could it spill?"
- "If this is 8+ points, should we talk about breaking it down?"

If the estimate seems off, reference the story-pointer scale:
- 1: Trivial (< half day)
- 3: Straightforward (1-2 days)
- 5: Medium (2-4 days)
- 8: Large, needs design doc (1+ week)
- 13: Too large -- must split

#### Activity Type Fit
- "The activity type is [type] -- does that match the work?"
- If missing: "This looks like [recommended type] based on the description. Sound right?"

Use these signals to recommend:
| Ticket Signals | Recommended Activity Type |
|----------------|--------------------------|
| Bug, defect, tech debt, toil | Quality / Stability / Reliability |
| New feature, product requirement | Product / Portfolio Work |
| Refactoring, tooling, automation | Future Sustainability |
| CVE, vulnerability, compliance | Security & Compliance |
| Escalation, outage, support | Incidents & Support |
| Training, onboarding, mentorship | Associate Wellness & Development |

#### Release Targeting
- If Fix Version is set: "This targets [version] — does that timeline still make sense?"
- If Fix Version is missing: "No Fix Version set. Is this tied to a specific release, or is it general backlog?"
- If the user wants to set a Fix Version, validate against `mcp__atlassian__jira_get_project_versions` before setting
- Set via `mcp__atlassian__jira_update_issue` with `fields: {"fixVersions": [{"name": "Version Name"}]}`

### Step 5: Fix Along the Way

Throughout the assessment, when issues are found:
1. Surface the concern clearly
2. Suggest a specific fix
3. Ask for confirmation before making changes
4. Make the change using `mcp__atlassian__jira_update_issue`

Common fixes:
- **Set story points**: `fields: {"customfield_10016": X}`
- **Set activity type**: `fields: {"customfield_10464": {"value": "Type Value"}}`
- **Update description**: `fields: {"description": "Updated description in Markdown"}`
- **Set component**: use the `components` parameter

**Description format**: Write descriptions in Markdown. The MCP server converts to JIRA's native format automatically.

### Step 6: Verdict

After walking through all areas (or the areas the user wanted to discuss), deliver a clear verdict:

- **Sprint-Ready**: Good to go. Scope is clear, criteria are testable, estimate makes sense, references are accurate.
- **Nearly Ready**: 1-2 minor things to sort. [List what's left.]
- **Needs Work**: Significant gaps. [List the key issues.]
- **Not Ready**: Scope is unclear, criteria are missing or untestable, the ticket is too large, or the underlying problem no longer exists.

Factor in Validate findings:
- Stale code references but otherwise good: **Nearly Ready** (needs description update)
- Underlying problem no longer exists (bug fixed, feature already implemented): **Not Ready** (may need to be closed rather than refined)
- Resolved dependency: informational (suggest removing the dependency note, doesn't affect readiness)

End with any remaining action items and offer to check the next ticket if the user has more.

## Integration

- **hygiene**: Mechanical field checks (complementary -- this skill runs them inline as Step 2)
- **set-activity-type**: Set or change activity type when identified as missing
- **story-pointer**: Detailed estimation if points need deeper analysis
- **ghostwriter**: Tone and style for all written suggestions

## Notes

- Do NOT use jira-cli or Bash for JIRA queries — use the mcp__atlassian__ MCP tools only
- Bash is used in Step 3 (Validate) for `gh` CLI operations only (GitHub link/PR checking)
- Code inspection tools (Read, Grep, Glob) are read-only. Never modify source code files during triage.
- The Validate step checks the local repo only. If a ticket references a different repository, note it and move on.
