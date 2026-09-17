---
name: todo
description: Manage todos in today's daily note, add items or mark them done.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__get_note_outline
  - mcp__obsidian__patch_note
  - mcp__obsidian__write_note
  - Bash
  - Read
argument-hint: 'add <item> | done <item>'
---

# Todo: Manage Daily Note Tasks

Add and complete todos in today's daily note. To start working on one, use `/webby:work`.

## Section

The daily note has one todo section, `#### To Dos`, which runs until `#### Log`. The Rollover Daily Todos plugin copies unchecked items forward when a new daily note is created; it does not watch the file afterwards, so adding and checking items here is safe.

## Subcommands

### `add <item>`

1. Get today's date via `date +%Y-%m-%d`
2. Read `10_Daily/YYYY-MM-DD.md` via `mcp__obsidian__read_note`. If it does not exist, create it (see Creating the daily note).
3. Confirm `#### To Dos` exists via `mcp__obsidian__get_note_outline`. If it is missing, append the item to the end of the note with `write_note` `mode: "append"` and tell the user the note is not using the standard template.
4. Extract the section content (from `#### To Dos` to `#### Log`)
5. Insert `- [ ] <item>` before the trailing blank `- [ ] ` placeholder if one exists, otherwise after the last existing item
6. Patch via `mcp__obsidian__patch_note` with the exact old section as `oldString` and the updated section as `newString`

**Sub-tasks:** `>` nests: `add Disaster recovery > Who owns this?` produces
```
- [ ] Disaster recovery
	- [ ] Who owns this?
```

**Multiple items:** comma or "and" separated arguments become separate `- [ ]` entries.

**Wiki-links:** if the item names a vault note, keep the `[[link]]` in the todo text. `/webby:work` uses it to pull context.

#### Patch Example

Current section:
```
#### To Dos
- [ ] MCP debug server for Hyperfleet
- [ ] 

#### Log
```

Adding "Review PR #55":
- `oldString`: `#### To Dos\n- [ ] MCP debug server for Hyperfleet\n- [ ] \n\n#### Log`
- `newString`: `#### To Dos\n- [ ] MCP debug server for Hyperfleet\n- [ ] Review PR #55\n- [ ] \n\n#### Log`

Output:
```
Added to To Dos:
- [ ] Review PR #55
```

### `done <item>`

1. Read today's daily note
2. Search `#### To Dos` for unchecked items matching the argument (case-insensitive, partial match is fine)
3. Exactly one match: replace `- [ ]` with `- [x]` via `mcp__obsidian__patch_note`
4. Multiple matches: list them numbered and ask which one
5. No match: say so and list the open items
6. Completing a parent item also completes its indented sub-tasks

Output:
```
Completed:
- [x] MCP debug server for Hyperfleet
```

## Creating the daily note

If `10_Daily/YYYY-MM-DD.md` does not exist, read `${CLAUDE_PLUGIN_ROOT}/skills/_shared/daily-template.md`, replace `YYYY-MM-DD` with today's date and `DayOfWeek, Month D, YYYY` with the output of `date +"%A, %B %-d, %Y"`, then write it with `mcp__obsidian__write_note`. Do not inline the template here; the shared file is the single source of truth and must match the vault's `80_System/Templates/Daily-Template.md`.

## Fallback

If `patch_note` fails, fall back to `mcp__obsidian__write_note` with `mode: "append"` and tell the user the entry was appended to the end of the file and needs manual repositioning.

## Notes

- Apply the ghostwriter skill for tone in any output
- Confirm what was done, don't repeat the entire daily note
- The blank `- [ ] ` placeholder at the end of the section is an Obsidian convention for easy manual entry. Preserve it (insert before it, not after)
