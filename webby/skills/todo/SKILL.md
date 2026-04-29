---
name: todo
description: Manage todos in today's daily note — add items, mark them done, or pick
  one to focus on.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__patch_note
  - mcp__obsidian__write_note
  - Bash
argument-hint: 'add <item> | done <item> | pick'
---

# Todo: Manage Daily Note Tasks

Add, complete, and pick todos from today's daily note. All operations target the single todo section in the daily note template.

## Section

The daily note has one todo section:

- `#### To Dos` — all tasks. Runs until `#### Log`. Managed by the Rollover Daily Todos plugin (rolls unchecked items forward between days). Adding and checking items is safe.

## Subcommands

### `add <item>` — Add a todo

1. Get today's date via `date +%Y-%m-%d`
2. Read `Daily/YYYY-MM-DD.md` via `mcp__obsidian__read_note`. If it doesn't exist, create it from the template (see Template section).
3. Target section is always `#### To Dos`
4. Extract the section content (from `#### To Dos` to `#### Log`)
5. Append `- [ ] <item>` before the trailing blank `- [ ] ` placeholder if one exists, or after the last existing item
6. Patch via `mcp__obsidian__patch_note` with the exact old section as `oldString` and the updated section as `newString`

**Sub-tasks:** If the argument uses `>` to indicate nesting, create indented sub-tasks:
- Input: `add Disaster recovery > Who owns this?`
- Result:
  ```
  - [ ] Disaster recovery
  	- [ ] Who owns this?
  ```

**Multiple items:** If the argument contains multiple items separated by commas or "and", add each as a separate `- [ ]` entry.

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

#### Output

```
Added to To Dos:
- [ ] Review PR #55
```

### `done <item>` — Mark a todo as complete

1. Read today's daily note
2. Search the todo section for unchecked items (`- [ ]`) matching the argument (case-insensitive, partial match is fine)
3. If exactly one match: replace `- [ ]` with `- [x]` via `mcp__obsidian__patch_note`
4. If multiple matches: list them numbered and ask the user which one
5. If no match: tell the user and list the open items so they can try again
6. When completing a parent item, also complete its indented sub-tasks

#### Patch Example

Marking "MCP debug" as done:
- `oldString`: `- [ ] MCP debug server for Hyperfleet`
- `newString`: `- [x] MCP debug server for Hyperfleet`

#### Output

```
Completed:
- [x] MCP debug server for Hyperfleet
```

### `pick` — Pick a task to focus on

1. Read today's daily note
2. Collect all unchecked `- [ ]` items from the todo section (skip blank placeholders)
3. Present them as a numbered list:

```
### Open Tasks

1. Disaster recovery
2. MCP debug server for Hyperfleet
3. Awesome Adapters :)
4. Sync with Christine
5. Sync with Michael
6. Talk with Rafael

Pick a number to start, or 0 to skip.
```

4. When the user picks one, log it to `#### Log` using the same patch pattern as session-log: `- Started: [item description]`

## Template for New Daily Notes

If the daily note does not exist, create it with pre-rendered content (replace date placeholders with actual values):

```markdown
---
date: YYYY-MM-DD
type: daily
tags:
  - daily
---
# DayOfWeek, Month D, YYYY

#### To Dos
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

## Fallback

If `patch_note` fails, fall back to `mcp__obsidian__write_note` with `mode: "append"` and tell the user the entry was appended to the end of the file and may need manual repositioning.

## Notes

- Apply the ghostwriter skill for tone in any output
- Keep output concise. Confirm what was done, don't repeat the entire daily note
- The blank `- [ ] ` placeholder at the end of the section is an Obsidian convention for easy manual entry. Preserve it when adding items (insert before it, not after)
