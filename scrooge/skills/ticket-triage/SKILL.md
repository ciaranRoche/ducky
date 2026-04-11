---
name: ticket-triage
description: Interactive triage session - collaboratively assess a ticket's validity, completeness, and sprint-readiness. Activates when users ask to "triage a ticket", "is this ticket ready?", or "walk me through this ticket".
argument-hint: <ticket-key>
---

# JIRA Ticket Triage

Interactive triage session. Walk through a ticket together, assess whether it's actually ready for sprint, and fix issues along the way.

This is not a field checklist -- that's `/ticket-hygiene`. This is a conversation about whether the ticket makes sense, has clear scope, and is ready to be picked up.

## Arguments
- `$1`: Ticket key (e.g., HYPERFLEET-123). If omitted, ask the user.

## Configuration

- `DUCKY_JIRA_PROJECT`: JIRA project key (default: `HYPERFLEET`)
- `JIRA_BASE_URL`: JIRA instance URL (default: `https://redhat.atlassian.net`)

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

Pull the ticket details and set the scene.

```bash
jira issue view TICKET-KEY --plain 2>/dev/null
```

```bash
jira issue view TICKET-KEY --raw 2>/dev/null
```

Present a brief summary:
- Title, type, status, assignee
- Story points (if set), component, activity type

Then ask a grounding question to start the conversation:
- "What's your take on this one -- does the scope feel right?"
- "How confident are you this is sprint-ready?"
- "Any context on this one that isn't in the description?"

Pick the question that fits -- if the ticket looks thin, lean toward scope. If it looks complete, lean toward confidence.

### Step 2: Quick Hygiene Check

Run the 6 required field checks inline (don't invoke ticket-hygiene, just check the fields from the data you already fetched):

1. Title: clear, actionable, under 100 characters
2. Description: detailed (> 100 characters)
3. Acceptance criteria: at least 2 testable criteria
4. Story points: set (scale: 0, 1, 3, 5, 8, 13)
5. Component: set and valid
6. Activity type: set for capacity planning

Report briefly: "Quick field check: 5/6 look good. Missing Activity Type. Want me to set that now, or dig into the content first?"

If everything passes: "Fields look clean -- let's talk about the content."

If there are mechanical gaps, offer to fix them immediately:
- Missing story points: "Want me to set this? What feels right -- 3? 5?"
- Missing activity type: "This looks like [type] to me -- sound right?"
- Missing component: Check valid components first, then suggest

### Step 3: Content Assessment

Work through these areas one at a time. Ask one question, wait for the user's response, then move to the next area. Skip areas the user is confident about.

#### Scope Clarity
- "Is the description clear enough that someone unfamiliar with the codebase could pick this up?"
- "Could you explain what 'done' looks like for this in one sentence?"
- "Are there implicit assumptions here that should be made explicit?"

If the description is thin or vague, suggest improvements. Offer to update the description (using JIRA wiki markup, written to a temp file).

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

### Step 4: Fix Along the Way

Throughout the assessment, when issues are found:
1. Surface the concern clearly
2. Suggest a specific fix
3. Ask for confirmation before making changes
4. Make the change using jira-cli

Common fixes:
```bash
# Set story points
jira issue edit TICKET-KEY --custom story-points=X --no-input

# Set activity type
jira issue edit TICKET-KEY --custom activity-type="Type Value" --no-input

# Update description (write to temp file first, then apply)
jira issue edit TICKET-KEY -b "$(cat /tmp/updated-description.txt)" --no-input
```

**JIRA Wiki Markup reminders** (for any description updates):
- Headers: `h3. Title` (space after period)
- Bullets: `* item`, nested: `** item`
- Bold: `*bold*`, Inline code: `{{code}}`
- NO curly braces `{}` in content
- Always write to temp file first

### Step 5: Verdict

After walking through all areas (or the areas the user wanted to discuss), deliver a clear verdict:

- **Sprint-Ready**: Good to go. Scope is clear, criteria are testable, estimate makes sense.
- **Nearly Ready**: 1-2 minor things to sort. [List what's left.]
- **Needs Work**: Significant gaps. [List the key issues.]
- **Not Ready**: Scope is unclear, criteria are missing or untestable, or the ticket is too large.

End with any remaining action items and offer to check the next ticket if the user has more.

## Prerequisites

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`

## Integration

- **ticket-hygiene**: Mechanical field checks (complementary -- this skill runs them inline)
- **sprint-hygiene**: Bulk audit of all sprint tickets
- **set-activity-type**: Set or change activity type when identified as missing
- **story-pointer**: Detailed estimation if points need deeper analysis
- **ghostwriter**: Tone and style for all written suggestions
