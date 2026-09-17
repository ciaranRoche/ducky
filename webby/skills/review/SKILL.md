---
name: review
description: Weekly review, roll the last five daily logs into a project-grouped summary
  and write it under "A day in review" on today's note. Run on Friday or whenever a week closes.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__read_multiple_notes
  - mcp__obsidian__list_directory
  - mcp__obsidian__get_note_outline
  - mcp__obsidian__patch_note
  - Bash
argument-hint: '[days, default 5]'
---

# Review: Weekly Roll-up

Read the last N daily logs (default 5 working days, including today), group by project, and write a short summary under `#### A day in review` in today's daily note. This is the one skill allowed to write that section.

## Behavior

1. **Resolve the window**
   - Today via `date +%Y-%m-%d`. N from the argument, default 5.
   - List `10_Daily/` with `mcp__obsidian__list_directory`, keep `.md`, drop `.sync-conflict-*`, sort descending, take the first N files with date <= today.
2. **Read them** with `mcp__obsidian__read_multiple_notes` and extract each `#### Log` section (same parsing as `/webby:standup`). Skip empty logs and say which days were empty.
3. **Group and condense**
   - Group by `[project]` prefix, "General" for unprefixed
   - Drop `- Started:` markers
   - Per project, condense to 2-5 bullets: what shipped, what was decided, what is still open
   - Keep `[[wiki-links]]` and ticket/PR references
4. **Present the draft** to the user before writing. Format:

```
### Week of YYYY-MM-DD to YYYY-MM-DD

_[hyperfleet]_
- ...

_[homelab]_
- ...

**Open threads:** ...
```

5. **On confirmation, write it** to today's note:
   - Confirm `#### A day in review` exists via `mcp__obsidian__get_note_outline`; if not, tell the user and stop (do not append elsewhere)
   - `oldString` is the exact block from `#### A day in review\n` to just before `#### Gratitude`
   - `newString` is that block with the summary inserted (preserve anything the user already wrote there, put the summary after it)
   - Patch with `mcp__obsidian__patch_note`
6. **Suggest follow-ups**: anything in "Open threads" that has no todo yet, offer `/webby:todo add` for it.

## Notes

- Apply the ghostwriter skill for tone
- Always show before writing, this section is the user's voice
- Never modifies any note other than today's
