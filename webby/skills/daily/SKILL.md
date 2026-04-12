---
name: daily
description: Read today's daily note from Obsidian — surface todos, context, and session
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
2. **Read the daily note** at path `Daily/YYYY-MM-DD.md` using `mcp__obsidian__read_note`
3. **If the note does not exist**, tell the user and offer to create it. Do not create it automatically.
4. **Parse and present** the following sections:

### Sections to Surface

- `#### What do I want to do today or tomorrow?` — present as **Today's Focus** (only unchecked `- [ ]` items)
- `#### Work To Dos` — present as **Open Work Items** (only unchecked `- [ ]` items, count them)
- `#### Log` — present as **Session History** (what's already been logged today, if anything)
- `#### Related Notes` — surface any linked projects, meetings, or learning notes

### Output Format

Present a clean, scannable summary. Do not dump the raw markdown. Example:

```
### Today (YYYY-MM-DD)

**Focus:**
- Item from "What do I want to do today or tomorrow?"

**Open Work Items:** (X remaining)
- [ ] Item 1
- [ ] Item 2

**Session History:**
- [ducky] Designed webby plugin architecture
- [hyperfleet] Reviewed adapter PR

**Related:** [[Project Note]], [[Meeting Note]]
```

If there are no items in a section, omit that section entirely.

## Notes

- The daily note path is always `Daily/YYYY-MM-DD.md` with no subdirectories
- Apply the ghostwriter skill for tone
- Keep the output brief. The goal is orientation, not a full readout
