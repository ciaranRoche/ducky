---
name: daily
description: Read today's daily note from Obsidian, surface todos, context, and session
  history. Use this to get oriented at the start of a session.
allowed-tools:
  - mcp__obsidian__read_note
  - Bash
argument-hint: ''
---

# Daily: Read Today's Context

Read today's daily note from the Obsidian vault and present a concise summary of what's on the agenda.

## Behavior

1. **Get today's date** using `date +%Y-%m-%d` via Bash
2. **Read the daily note** at path `10_Daily/YYYY-MM-DD.md` using `mcp__obsidian__read_note`
3. **If the note does not exist**, tell the user and suggest `/webby:todo add <item>` (which creates it). Do not create it here.
4. **Parse and present** the following sections:

### Sections to Surface

- `#### To Dos`: present as **Open Items** (only unchecked `- [ ]` items, count them, skip the blank placeholder)
- `#### Log`: present as **Session History** (what's already been logged today, if anything)
- `#### Related Notes`: surface any wiki-links under Projects / Meetings / Learning

### Output Format

Present a clean, scannable summary. Do not dump the raw markdown. Example:

```
### Today (YYYY-MM-DD)

**Open Items:** (2 remaining)
- [ ] Item 1
- [ ] Item 2

**Session History:**
- [ducky] Designed webby plugin architecture
- [hyperfleet] Reviewed adapter PR

**Related:** [[Project Note]], [[Meeting Note]]
```

If there are no items in a section, omit that section entirely. If there are open items, end with: `Run /webby:work to pick one.`

## Notes

- The daily note path is always `10_Daily/YYYY-MM-DD.md` with no subdirectories
- Apply the ghostwriter skill for tone
- Read-only. Keep the output brief, the goal is orientation, not a full readout
