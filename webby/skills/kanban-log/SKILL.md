---
name: kanban-log
description: Log activity to a kanban task note — capture progress, decisions, and
  status updates for the active task.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__patch_note
  - mcp__obsidian__write_note
  - Bash
argument-hint: '<task name> [summary]'
---

# Kanban Log: Per-Task Activity Logging

Append structured log entries to a kanban task note's `#### Log` section. This is the kanban equivalent of `session-log` — but instead of logging to the daily note, it logs to a specific task's note.

## Board Reference

- **Board path:** `Work/Kanban.md`
- **Task notes folder:** read from the board's `new-note-folder` setting (default: `Work/Tasks/`)

## Behavior

1. **Identify the target task.** The user provides a task name as the first argument. If omitted, infer from the current session context (e.g., if the user has been working on a specific task this session).

2. **Locate the task note.**
   - Read the kanban board at `Work/Kanban.md` using `mcp__obsidian__read_note`
   - Parse the settings block to extract `new-note-folder`
   - Construct the note path: `<new-note-folder>/<task name>.md`

3. **Read the task note** using `mcp__obsidian__read_note`.

4. **If the note does not exist**, create it using `mcp__obsidian__write_note` with the template below, then proceed to log.

5. **Get today's date** using `date +%Y-%m-%d` via Bash.

6. **Generate log entries** from the conversation context:
   - Each entry: `- YYYY-MM-DD: <concise outcome>`
   - Focus on outcomes, decisions made, blockers hit
   - Skip trivial actions (file reads, exploratory searches that led nowhere)
   - If the user provides a summary as the second argument, use that instead of auto-generating

7. **Patch the `#### Log` section** using `mcp__obsidian__patch_note`:
   - Extract the exact content between `#### Log` and `#### Links`
   - If the log section only contains the placeholder (`- `), replace it with the new entry
   - Otherwise, append the new entries after the last existing entry

   Patch example — first log entry (replacing placeholder):
   - `oldString`: `#### Log\n- \n\n#### Links`
   - `newString`: `#### Log\n- 2026-04-16: Researched API design patterns, decided on OpenAPI 3.1\n\n#### Links`

   Patch example — appending to existing entries:
   - `oldString`: `#### Log\n- 2026-04-14: Started research\n\n#### Links`
   - `newString`: `#### Log\n- 2026-04-14: Started research\n- 2026-04-16: Drafted initial endpoints, shared with team\n\n#### Links`

8. **Report** what was logged:
   ```
   Logged to [[Design API spec]]:
   - 2026-04-16: Drafted initial endpoints, shared with team
   ```

**Fallback:** If `patch_note` fails, use `mcp__obsidian__write_note` with `mode: "append"` and tell the user the entry was appended to the end of the file.

## Task Note Template

If a task note does not exist, create it with:

```markdown
#task
### What
<task name>

### Why

### How

### Status
- Created: YYYY-MM-DD

#### Log
- 

#### Links
```

## Log Entry Guidelines

- One line per entry, concise, action-oriented
- Past tense describing what was accomplished
- Include key decisions or blockers inline
- Use `[[wiki-links]]` to reference other vault notes when relevant

Good examples:
```
- 2026-04-14: Researched auth patterns, decided on OAuth2 with PKCE
- 2026-04-15: Implemented token refresh flow, hit CORS issue with redirect
- 2026-04-16: Fixed CORS via proxy config, PR ready for review
```

Bad examples:
```
- 2026-04-14: Worked on auth stuff
- 2026-04-15: Read 12 files, searched for 5 patterns, eventually found the issue on line 42 of auth.go...
```

## Notes

- Apply the ghostwriter skill for tone
- Keep entries concise — one line per meaningful outcome
- Do NOT modify the kanban board itself — this skill only operates on task notes
