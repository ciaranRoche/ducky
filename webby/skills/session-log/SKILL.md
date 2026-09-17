---
name: session-log
description: Append a structured session summary to today's daily note under the Log
  section. Run this at the end of a session to capture what was done.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__get_note_outline
  - mcp__obsidian__patch_note
  - mcp__obsidian__write_note
  - Bash
  - Read
argument-hint: '[optional summary]'
---

# Session Log: Capture Session to Daily Note

Append a structured summary of the current session's work to today's daily note under `#### Log`.

## Behavior

1. **Get today's date** using `date +%Y-%m-%d` via Bash
2. **Read today's daily note** at `10_Daily/YYYY-MM-DD.md` using `mcp__obsidian__read_note`
3. **If the note does not exist**, create it first (see Creating the daily note), then continue
4. **Confirm `#### Log` exists** via `mcp__obsidian__get_note_outline`. If it is missing, append the entries to the end of the note with `write_note` `mode: "append"` and tell the user the note is not on the standard template.
5. **Generate log entries** from the conversation context:
   - One bullet per meaningful outcome
   - Prefix with `[project-name]` when working in a specific repo (use the directory name)
   - Focus on outcomes: what was done, decisions made, meaningful discoveries
   - Skip trivial actions (file reads, dead-end searches)
   - If the user provides a summary as an argument, use that instead
   - If a vault note was created or updated this session, reference it with a `[[wiki-link]]` in the entry
6. **Patch the section** using `mcp__obsidian__patch_note`:
   - `oldString` is the exact block from `#### Log\n` to just before `#### A day in review`
   - `newString` is the same block with the new entries appended
   - If the section only contains the placeholder `- `, replace it

### Patch Example

Current:
```
#### Log
- 

#### A day in review
```
- `oldString`: `#### Log\n- \n\n#### A day in review`
- `newString`: `#### Log\n- [ducky] Built webby plugin for Obsidian vault integration\n\n#### A day in review`

With existing entries:
- `oldString`: `#### Log\n- [hyperfleet] Reviewed adapter PR #52\n\n#### A day in review`
- `newString`: `#### Log\n- [hyperfleet] Reviewed adapter PR #52\n- [ducky] Built webby plugin\n\n#### A day in review`

### Fallback

If `patch_note` fails (content changed between read and patch), fall back to `mcp__obsidian__write_note` with `mode: "append"` and tell the user the entries were appended to the end of the file and need moving under `#### Log`.

## Log Entry Format

- One line, past tense, action-oriented
- `[project-name]` prefix when applicable (this is what `/webby:standup` and `/webby:review` group on)
- Key decisions or next steps inline where useful

Good:
```
- [hyperfleet] Reviewed adapter status aggregation PR, left feedback on error handling edge case
- [ducky] Designed webby plugin architecture, see [[Webby Plugin Design]]
- Researched MCP server options for cross-project vault access, chose MCPVault
- [homelab] Configured Ambient mesh on NUC cluster, hit DNS resolution issue with eastwest gateway
```

Bad:
```
- Worked on stuff
- Read 15 files, searched for 3 patterns, found the bug in line 42 of server.go where ...
```

## Creating the daily note

Read `${CLAUDE_PLUGIN_ROOT}/skills/_shared/daily-template.md`, replace `YYYY-MM-DD` with today's date and `DayOfWeek, Month D, YYYY` with `date +"%A, %B %-d, %Y"`, then write it with `mcp__obsidian__write_note`. Do not inline the template here.

## Notes

- Apply the ghostwriter skill for tone
- The daily note is a reference, not a transcript
