---
name: pr-feedback
description: Gather open review feedback on a PR you own and step through each thread
  interactively, fixing code or replying as you go. Activates when users ask to "address
  feedback", "work through PR comments", or "resolve PR feedback".
allowed-tools:
  - Bash
  - Read
  - Edit
  - Grep
  - Glob
  - mcp__writing-samples__qdrant-find
argument-hint: [PR-URL-or-number]
---

# PR Feedback

Fetch unresolved review threads on a PR you own, then step through each one interactively. Fix code, reply to comments, or skip -- one thread at a time.

## Arguments
- `$1` (optional): PR URL or number. If omitted, detects the PR for the current branch.

## When to Use This Skill

- "address feedback on my PR"
- "work through PR comments"
- "resolve PR feedback"
- "step through review comments"
- "what feedback do I have on my PR?"

## Interaction Style

Follow the conversational patterns from the pair-programmer and ticket-triage skills:
- **Short responses**: 1-3 sentences per turn to keep momentum
- **One question at a time**: Present one thread, ask how to handle it, wait
- **User-driven pacing**: If they say "skip", move on immediately
- **Casual tone**: Use ghostwriter skill for voice
- **Fix as you go**: Make code changes when asked, confirm before editing
- **No lecturing**: Present the feedback, ask what to do, execute

## Instructions

### Step 1: Fetch PR and verify ownership

```bash
gh pr view ${1:+$1} --json number,title,state,author,url,headRefName 2>/dev/null
```

```bash
gh api user --jq '.login' 2>/dev/null
```

- If no PR found, inform the user and suggest `gh pr list`.
- If the current user is not the PR author, note this: "Heads up, this PR belongs to @[author]. This skill is designed for working through feedback on your own PRs, but we can still look at the threads if you want."
- Present a brief summary: PR number, title, branch.

### Step 2: Fetch unresolved review threads

Get owner and repo:

```bash
gh repo view --json owner,name --jq '"\(.owner.login)/\(.name)"' 2>/dev/null
```

Fetch review threads via GraphQL:

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!, $number: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $number) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          isOutdated
          path
          line
          startLine
          comments(first: 50) {
            nodes {
              id
              databaseId
              author { login }
              body
              createdAt
              url
            }
          }
        }
      }
    }
  }
}' -F owner='OWNER' -F repo='REPO' -F number=NUMBER
```

Replace `OWNER`, `REPO`, `NUMBER` with the values from Step 1.

Filter to **unresolved threads only** (`isResolved: false`). Sort by file path so you work through the PR file-by-file.

Present a summary:
- "Found **X unresolved threads** across **Y files** from **Z reviewers**. Let's step through them."
- If outdated threads exist, note how many: "N of these are on code that's changed since the comment was left."

If zero unresolved threads: "No open feedback -- you're clean. Nice work." Stop here.

### Step 3: Step through each thread (one at a time)

For each unresolved thread, in file-path order:

#### 3a. Present the feedback

Show:
- **File and line**: `path/to/file.go:42`
- **Reviewer**: @reviewer-name
- **Comment thread**: All messages in the conversation, in order
- **Outdated flag**: If the thread is outdated, note: "(outdated -- code has changed since this comment)"
- **Link**: URL to the comment on GitHub

#### 3b. Show code context

Use the `Read` tool to show ~10 lines around the mentioned line in the current version of the file. If the thread has a `startLine`, show from `startLine` to `line`.

If the file or line no longer exists (refactored, deleted), note that.

#### 3c. Ask how to handle it

Ask one question and wait:

> How do you want to handle this one?
> - **Fix** -- make a code change
> - **Reply** -- respond to the reviewer
> - **Already fixed** -- this is already addressed, just need to say so
> - **Skip** -- come back to it later

Don't list all four options every time -- read the situation. If the feedback is clearly a valid bug, lean toward "Want me to fix this?" If it's a style preference, lean toward "Want to reply or just skip?"

#### 3d. Execute the action

**Fix:**
1. Understand the feedback and the relevant code (use `Read`, `Grep`, `Glob` as needed)
2. Make the change using `Edit`
3. After fixing, ask: "Want me to reply letting them know this is fixed?"
4. If yes, draft a short reply in ghostwriter tone and post it

**Reply:**
1. Discuss the response with the user if needed
2. Draft the reply using ghostwriter tone (use `mcp__writing-samples__qdrant-find` for style calibration if available)
3. Show the draft to the user for approval
4. Post using `gh api`:

```bash
gh api repos/OWNER/REPO/pulls/NUMBER/comments \
  -f body="reply text" \
  -F in_reply_to=FIRST_COMMENT_DATABASE_ID
```

Where `FIRST_COMMENT_DATABASE_ID` is the `databaseId` of the first comment in the thread.

**Already fixed:**
1. Draft a brief reply: "Good catch, this is already addressed in [commit/change]" (ghostwriter tone)
2. Post the reply

**Skip:**
1. Move to the next thread silently
2. Track skipped threads for the summary

#### 3e. Offer to resolve the thread

After fixing or replying, ask: "Want me to resolve this thread?"

If yes:

```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: {threadId: $threadId}) {
    thread { isResolved }
  }
}' -F threadId='THREAD_NODE_ID'
```

Where `THREAD_NODE_ID` is the `id` from the GraphQL query in Step 2.

Then move to the next thread.

### Step 4: Summary

After all threads are processed (or the user says "done" / "that's enough"):

```
### Feedback Summary

| Action        | Count |
|---------------|-------|
| Fixed         | X     |
| Replied       | Y     |
| Already fixed | Z     |
| Skipped       | W     |

[If code changes were made:]
You've got local changes -- don't forget to commit and push.

[If all threads addressed:]
All threads addressed. You might want to re-request review.

[If threads were skipped:]
N threads still open -- run this again when you're ready for the rest.
```

## Notes

- This skill is for working through feedback on PRs you own. It complements the `review` pipeline which is for reviewing others' PRs.
- Always use ghostwriter tone when drafting replies.
- Don't resolve threads without asking -- the user might want to leave them open for the reviewer to verify.
- If the user says "done" or "that's enough" mid-way through, jump to the summary.
- Track the tally (fixed/replied/skipped) throughout the session for the final summary.
