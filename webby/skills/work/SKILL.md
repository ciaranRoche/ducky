---
name: work
description: Pick a todo from today's daily note and start working on it with Claude.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__patch_note
  - mcp__obsidian__search_notes
  - Bash
argument-hint: ''
---

# Work: Pick a Todo and Start

Pick an open todo from today's daily note or the kanban board and begin working on it. Bridges Obsidian into an active Claude work session.

## Phase 1 — Select

1. **Get today's date** using `date +%Y-%m-%d` via Bash
2. **Read today's daily note** at `Daily/YYYY-MM-DD.md` using `mcp__obsidian__read_note`
3. **Read the kanban board** at `Work/Kanban.md` using `mcp__obsidian__read_note`. If the board does not exist, skip kanban items silently.
4. **Collect tasks from both sources:**

   **Daily note** — unchecked `- [ ]` items from:
   - `#### What do I want to do today or tomorrow?` (personal)
   - `#### Work To Dos` (work)
   - Skip blank placeholders (items where the text after `- [ ] ` is empty or whitespace-only)

   **Kanban board** — parse columns by splitting on `## ` headings (content before `%% kanban:settings`):
   - Collect unchecked `- [ ]` items from the "In Progress" column first (already active)
   - Then collect unchecked `- [ ]` items from the first column (Backlog)

5. **If no items from either source**, tell the user and suggest `/webby:todo add` or `/webby:kanban add`
6. **Present as a numbered list** with source labels:

```
### Open Tasks

**Daily Note:**
1. Buy groceries
2. Disaster recovery

**Kanban — In Progress:**
3. [[DB Long Transactions]]

**Kanban — Backlog:**
4. [[IC Team Setup]]
5. [[Design API spec]]

Pick a number to start working on.
```

7. Wait for the user to pick a number

## Phase 2 — Ground

Once the user selects a task:

1. **Log the selection** to today's daily note `#### Log` using `mcp__obsidian__patch_note`:
   - Extract the exact content between `#### Log` and `#### A day in review`
   - Append `- Started: [item description]`
   - If the log section only contains the placeholder (`- `), replace it

   Patch example — current section:
   ```
   #### Log
   - [hyperfleet] Reviewed adapter PR
   
   #### A day in review
   ```

   Logging "Started: Disaster recovery":
   - `oldString`: `#### Log\n- [hyperfleet] Reviewed adapter PR\n\n#### A day in review`
   - `newString`: `#### Log\n- [hyperfleet] Reviewed adapter PR\n- Started: Disaster recovery\n\n#### A day in review`

   **Fallback:** If `patch_note` fails, fall back to `mcp__obsidian__write_note` with `mode: "append"` and tell the user the entry was appended to the end of the file.

2. **If the selected task is from the kanban board:**
   - If the task is not already in "In Progress", move it there by patching `Work/Kanban.md` (remove from source column, add to In Progress — same patch logic as `/webby:kanban move`)
   - Read the linked task note if it exists (parse `new-note-folder` from board settings, look for `<new-note-folder>/<task name>.md`). Surface `### What` and `#### Log` sections as context.
   - Log `- YYYY-MM-DD: Started` to the task note's `#### Log` section using `mcp__obsidian__patch_note`

3. **Search the vault for context** using `mcp__obsidian__search_notes` with the selected item's text as the query (`searchContent: true`, limit 5). If relevant notes are found:
   - Read the top 2-3 most relevant notes
   - Surface a brief summary (2-3 sentences max) with `[[wiki-links]]`
   - Only show notes that are clearly related — skip tangential matches
   - If nothing relevant is found, skip this step silently

4. **Ask one grounding question** tailored to the task type:
   - **Vague task** (e.g., "Disaster recovery", "Refactor auth"): "What specifically needs to happen here? Is this research, writing, code, or something else?"
   - **References a ticket or PR** (e.g., "Review PR #55", "JIRA-123"): "Want me to pull that up and get started?"
   - **Clearly actionable** (e.g., "Fix DNS resolution in mesh config"): "Any specific context or constraints I should know before diving in?"

5. Wait for the user's response

## Phase 3 — Execute

Based on the user's clarification, begin working on the selected task as the primary focus for this session. Use whatever tools, skills, and approaches are appropriate for the work. The skill does not prescribe execution — it bridges Obsidian into action.

## Notes

- Apply the ghostwriter skill for tone in all output
- Keep Phase 1 and Phase 2 output concise — the goal is to get into the work quickly, not to produce a report
- Phase 2 vault search is a lightweight context check, not a deep research pass. If deeper research is needed, that happens in Phase 3
