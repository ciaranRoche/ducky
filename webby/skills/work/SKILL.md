---
name: work
description: Pick a todo from today's daily note and start working on it with Claude.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__get_note_outline
  - mcp__obsidian__patch_note
  - mcp__obsidian__write_note
  - mcp__obsidian__search_notes
  - Bash
argument-hint: '[number or item text]'
---

# Work: Pick a Todo and Start

Pick an open todo from today's daily note and begin working on it. Bridges Obsidian into an active Claude work session.

## Phase 1: Select

1. **Get today's date** using `date +%Y-%m-%d` via Bash
2. **Read today's daily note** at `10_Daily/YYYY-MM-DD.md` using `mcp__obsidian__read_note`. If it does not exist, say so and suggest `/webby:todo add <item>`.
3. **Collect unchecked `- [ ]` items** from `#### To Dos`, skipping blank placeholders. Indented sub-tasks are listed under their parent, not as separate picks.
4. **If there are no items**, say so and suggest `/webby:todo add`
5. **If an argument was given**, match it (number, or case-insensitive partial text) and skip to Phase 2. Otherwise present a numbered list and wait:

```
### Open Tasks

1. Disaster recovery
2. Review [[ADR-0020 - Multitenancy]]
3. Sync with Christine

Pick a number to start working on.
```

## Phase 2: Ground

1. **Log the selection** to `#### Log` in today's daily note:
   - Confirm `#### Log` exists via `mcp__obsidian__get_note_outline`. If not, append with `write_note` `mode: "append"` and tell the user.
   - Extract the exact content between `#### Log` and `#### A day in review`
   - Append `- Started: <item description>`; if the section only holds the `- ` placeholder, replace it
   - Patch with `mcp__obsidian__patch_note`

   Example, current section:
   ```
   #### Log
   - [hyperfleet] Reviewed adapter PR

   #### A day in review
   ```
   - `oldString`: `#### Log\n- [hyperfleet] Reviewed adapter PR\n\n#### A day in review`
   - `newString`: `#### Log\n- [hyperfleet] Reviewed adapter PR\n- Started: Disaster recovery\n\n#### A day in review`

2. **Pull context from the vault:**
   - If the todo contains a `[[wiki-link]]`, read that note via `mcp__obsidian__read_note` and surface its outline (`mcp__obsidian__get_note_outline`) plus a 2-3 sentence summary. This is the primary context; skip the search below unless the note is thin.
   - Otherwise, `mcp__obsidian__search_notes` with the item text (`searchContent: true`, limit 5). Read the top 2-3 clearly relevant notes and summarise in 2-3 sentences with `[[wiki-links]]`. If nothing is relevant, skip silently.

3. **Ask one grounding question** tailored to the task type:
   - **Vague** ("Disaster recovery", "Refactor auth"): "What specifically needs to happen here? Research, writing, code, or something else?"
   - **References a ticket or PR** ("Review PR #55", "HYPERFLEET-123"): "Want me to pull that up and get started?"
   - **Clearly actionable** ("Fix DNS resolution in mesh config"): "Any context or constraints before I dive in?"

4. Wait for the user's response

## Phase 3: Execute

Work on the selected task as the primary focus of the session using whatever tools and skills fit. When done, `/webby:todo done <item>` and `/webby:session-log` close the loop; if the work produced something worth keeping, `/webby:vault-save` files it and links it from the daily note.

## Notes

- Apply the ghostwriter skill for tone
- Keep Phase 1 and 2 output concise, the goal is to get into the work quickly
- Phase 2 is a lightweight context check, not a research pass
