---
name: kanban
description: Manage kanban board tasks — add cards, move between columns, mark done,
  list, and reorder.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__patch_note
  - mcp__obsidian__write_note
  - Bash
argument-hint: 'add <item> | move <item> <column> | done <item> | list | rank <item>
  <position>'
---

# Kanban: Board Task Management

Manage a single kanban board in the Obsidian vault. Add cards, move them between columns, mark them done, and reorder.

## Board Location

- **Board path:** `Work/Kanban.md`
- **Task notes:** stored in the folder specified by the board's `new-note-folder` setting (default: `Work/Tasks/`)

## Board Format

The Obsidian Kanban plugin stores boards as markdown. Columns are `## Headings`, cards are `- [ ]` / `- [x]` list items. A settings block at the end stores board configuration.

## Parsing Logic

Every subcommand must read and parse the board before operating. Follow these steps:

1. Read the board at `Work/Kanban.md` using `mcp__obsidian__read_note`. The response has `fm` (frontmatter) and `content`.
2. **If the board does not exist**, offer to create it (see Board Initialization).
3. Split `content` at `%% kanban:settings` to separate the board body from the settings block.
4. Parse the settings block to extract `new-note-folder` (used for task note creation).
5. Split the board body on `## ` to extract columns. Each column has:
   - A name (the heading text)
   - A list of cards: lines matching `- [ ] ` or `- [x] `
6. Handle multi-line cards: if a line starts with whitespace (tab or spaces) after a card line, it is a continuation of that card.

## Subcommands

### `list` (default when no subcommand given)

1. Get today's date via `date +%Y-%m-%d` (Bash)
2. Read and parse the board
3. Present a clean summary of each column:

```
### Kanban

**Backlog** (3)
- [ ] [[Design API spec]]
- [ ] [[IC Team Setup]]
- [ ] Review DDR document

**In Progress** (1)
- [ ] [[DB Long Transactions]]

**Review** (0)

**Done** (2)
- [x] [[Handle CVE's]]
  ... and 1 more
```

4. Collapse columns with more than 5 cards (show first 5, then "... and N more")
5. If all columns are empty, say so and suggest `/webby:kanban add`

### `add <item> [to <column>]`

1. Get today's date via `date +%Y-%m-%d` (Bash)
2. Read and parse the board
3. Determine the target column:
   - Default: first column (Backlog)
   - If user specifies a column (e.g., "add X to In Progress"), match case-insensitively against column headings
   - If no match, list available columns and ask
4. Build the card line:
   - For task-like items (multi-word, not a URL): `- [ ] [[<item>]]`
   - For URLs or quick reminders: `- [ ] <item>`
   - Append `@{YYYY-MM-DD}` if the user provides a due date
   - Append any `#tags` the user specifies
5. Patch the board to insert the card at the bottom of the target column:

   **Patch strategy:** Find the target column's content (from `## Column Name` through to the blank lines before the next `## ` heading or `%% kanban:settings`). Insert the new card line after the last existing card (or after the heading if the column is empty).

   Patch example — adding to a column with existing cards:
   - `oldString`: `## Backlog\n\n- [ ] [[Existing Task]]\n`
   - `newString`: `## Backlog\n\n- [ ] [[Existing Task]]\n- [ ] [[New Task]]\n`

   Patch example — adding to an empty column:
   - `oldString`: `## Backlog\n\n\n`
   - `newString`: `## Backlog\n\n- [ ] [[New Task]]\n\n`

6. **Create a linked task note** if the card uses `[[wikilinks]]`:
   - Read `new-note-folder` from the board settings (default: `Work/Tasks`)
   - Check if the note already exists at `<new-note-folder>/<item>.md` using `mcp__obsidian__read_note`. If it exists, skip creation.
   - Create the note using `mcp__obsidian__write_note`:

   ```markdown
   #task
   ### What
   <item description or left blank for user to fill>

   ### Why

   ### How

   ### Status
   - Created: YYYY-MM-DD

   #### Log
   - 

   #### Links
   ```

7. Report what was done:
   ```
   Added to Backlog:
   - [ ] [[Design API spec]]

   Created task note: Work/Tasks/Design API spec.md
   ```

**Fallback:** If `patch_note` fails, read the full board content, modify it in memory, and use `mcp__obsidian__write_note` with `mode: "overwrite"` to rewrite the board. Warn the user.

### `move <item> <column>`

1. Read and parse the board
2. Find the card matching `<item>`:
   - Match case-insensitively against card text
   - Strip `[[` `]]`, `#tags`, and `@{dates}` from card text before matching
   - Partial matches are OK (e.g., "api spec" matches `- [ ] [[Design API spec]]`)
3. If no match: list all cards across all columns
4. If multiple matches: present numbered list and ask user to pick
5. Identify the source column (where the card currently is) and target column (matched case-insensitively against column headings)
6. If source and target are the same column, tell the user and stop

7. **Patch the board** with a single patch covering both the source and target columns:

   Determine which column appears first in the document. Build the `oldString` spanning from the earlier column's `## Heading` through the later column's card list. Build the `newString` with the card removed from the source and appended to the target.

   Patch example — moving from Backlog to In Progress:
   - `oldString`: `## Backlog\n\n- [ ] [[Task A]]\n- [ ] [[Task B]]\n\n\n## In Progress\n\n- [ ] [[Task C]]\n`
   - `newString`: `## Backlog\n\n- [ ] [[Task A]]\n\n\n## In Progress\n\n- [ ] [[Task C]]\n- [ ] [[Task B]]\n`

8. Report the move:
   ```
   Moved to In Progress:
   - [ ] [[Task B]]
   ```

**Fallback:** If `patch_note` fails (e.g., board was modified between read and patch), re-read the board, rebuild the patch, and retry once. If it fails again, fall back to full rewrite.

### `done <item>`

1. Read and parse the board
2. Find the card matching `<item>` (same matching logic as `move`)
3. Move the card to the "Done" column (same patch logic as `move`)
4. Toggle the card's checkbox: change `- [ ]` to `- [x]`

5. Report:
   ```
   Completed:
   - [x] [[Design API spec]]

   Moved from In Progress -> Done
   ```

### `rank <item> <position>`

1. Read and parse the board
2. Find the card matching `<item>` and identify its current column
3. Parse `<position>`:
   - A number (1-based index within the column)
   - `top` — alias for position 1
   - `bottom` — alias for last position
4. Remove the card from its current position in the column
5. Insert it at the target position
6. Patch the column's card list:
   - `oldString`: the full card list within the column (all `- [ ]` / `- [x]` lines)
   - `newString`: the reordered card list

7. Report:
   ```
   Ranked [[Design API spec]] to #1 in Backlog
   ```

## Board Initialization

If the board does not exist at `Work/Kanban.md`, offer to create it. If the user agrees, create it using `mcp__obsidian__write_note`:

```markdown
---

kanban-plugin: board

---

## Backlog



## In Progress



## Review



## Done



%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false,false],"new-note-folder":"Work/Tasks"}
```
%%
```

Use the exact frontmatter format above (blank lines around `kanban-plugin: board`). This is the format the Obsidian Kanban plugin expects.

## Notes

- Apply the ghostwriter skill for tone in all output
- Keep output concise — report what was done, not a verbose explanation
- Preserve exact whitespace when patching. Always read the board immediately before patching and use the raw content for `oldString` construction.
- Do NOT use jira-cli, Bash for vault operations, or any non-MCP approach to modify vault files
