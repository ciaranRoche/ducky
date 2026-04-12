---
name: session-log
description: Append a structured session summary to today's daily note under the Log
  section. Run this at the end of a session to capture what was done.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__patch_note
  - mcp__obsidian__write_note
  - Bash
argument-hint: '[optional summary]'
---

# Session Log: Capture Session to Daily Note

Append a structured summary of the current session's work to today's daily note under `#### Log`.

## Behavior

1. **Get today's date** using `date +%Y-%m-%d` via Bash
2. **Read today's daily note** at `Daily/YYYY-MM-DD.md` using `mcp__obsidian__read_note`
3. **If the note does not exist**, create it first using `mcp__obsidian__write_note` with pre-rendered template content (see Template section below). Then proceed with the patch.
4. **Generate log entries** from the conversation context:
   - Each entry is a single bullet point
   - Prefix with `[project-name]` if working in a specific repo (use the directory name)
   - Focus on outcomes: what was done, decisions made, meaningful discoveries
   - Skip trivial actions (file reads, exploratory searches that led nowhere)
   - If the user provides a summary as an argument, use that instead of auto-generating
5. **Locate the `#### Log` section** in the note content. The section runs from `#### Log` to the next `####` heading (`#### A day in review`).
6. **Patch the section** using `mcp__obsidian__patch_note`:
   - Extract the exact current content between `#### Log` and `#### A day in review`
   - Build `oldString` as that exact block (including the leading `#### Log\n` and trailing whitespace before `#### A day in review`)
   - Build `newString` as the same block with the new entries appended
   - If the log section only contains the placeholder (`- `), replace it with the new entries

### Patch Example

If the current section is:
```
#### Log
- 

#### A day in review
```

And the new entry is `- [ducky] Built webby plugin for Obsidian vault integration`, the patch would be:

- `oldString`: `#### Log\n- \n\n#### A day in review`
- `newString`: `#### Log\n- [ducky] Built webby plugin for Obsidian vault integration\n\n#### A day in review`

If the section already has entries:
```
#### Log
- [hyperfleet] Reviewed adapter PR #52

#### A day in review
```

Then:
- `oldString`: `#### Log\n- [hyperfleet] Reviewed adapter PR #52\n\n#### A day in review`
- `newString`: `#### Log\n- [hyperfleet] Reviewed adapter PR #52\n- [ducky] Built webby plugin\n\n#### A day in review`

### Fallback

If `patch_note` fails (e.g., content was modified between read and patch), fall back to `mcp__obsidian__write_note` with `mode: "append"` and tell the user the entries were appended to the end of the file and may need manual repositioning under `#### Log`.

## Log Entry Format

Each entry should be:
- One line, concise, action-oriented
- Prefixed with `[project-name]` when applicable
- Past tense describing what was accomplished
- Optionally include key decisions or next steps inline

Good examples:
```
- [hyperfleet] Reviewed adapter status aggregation PR, left feedback on error handling edge case
- [ducky] Designed webby plugin architecture for Obsidian vault integration
- Researched MCP server options for cross-project vault access, chose MCPVault
- [homelab] Configured Ambient mesh on NUC cluster, hit DNS resolution issue with eastwest gateway
```

Bad examples (too vague or too detailed):
```
- Worked on stuff
- Read 15 files, searched for 3 patterns, found the bug in line 42 of server.go where the error handling was missing a nil check on the response body reader which caused a panic when the upstream service returned a 502
```

## Template for New Daily Notes

If the daily note does not exist, create it with this pre-rendered content (replace YYYY-MM-DD and the full date string with the actual date):

```markdown
---
date: YYYY-MM-DD
type: daily
tags:
  - daily
---
# DayOfWeek, Month D, YYYY

#### What do I want to do today or tomorrow?
- [ ] 

#### Work To Dos
- [ ] 

#### Log
- 

#### A day in review

#### Gratitude

#### Related Notes
- Projects: 
- Meetings: 
- Learning: 
```

Use Bash to compute the full date string: `date +"%A, %B %-d, %Y"`.

## Notes

- Apply the ghostwriter skill for tone in the log entries
- Keep entries concise. The daily note is a reference, not a transcript
