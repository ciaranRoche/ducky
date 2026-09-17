# webby

Obsidian second brain. Daily notes, todos, session logging, standup, weekly review, vault search, and knowledge capture.

## Project Structure

```
skills/          Slash commands and reusable capabilities (SKILL.md files)
skills/_shared/  Shared assets referenced by skills (not a skill itself)
.claude-plugin/  Plugin manifest
```

## Key Conventions

- **Ghostwriter is central.** All vault entries and written output should use the ghostwriter skill for tone and structure.
- **The webby skill** (`user-invocable: false`) defines the persona and the daily-note workflow. It activates as background context for all vault operations.
- **MCP:** The `obsidian` MCP server is configured at user scope (run `/webby:vault-setup` if missing). There is no `.mcp.json` in the plugin, so no vault path is hardcoded. The `writing-samples` server (from the `ducky` plugin) is optional.
- **Daily notes** live at `10_Daily/YYYY-MM-DD.md`. The canonical structure is `skills/_shared/daily-template.md`, which must stay identical in headings to the vault's `80_System/Templates/Daily-Template.md`. Every skill anchors on those `####` headings: `To Dos`, `Log`, `A day in review`, `Gratitude`, `Related Notes`.
- **Section patching:** confirm the heading exists with `get_note_outline` before `patch_note`. If it is missing, append and tell the user. Never fall back silently.
- **No task board.** Kanban support was removed in 2.0.0. Work is tracked in the daily note's `#### To Dos`.
- **Vault structure:** `_AGENT_MANIFEST.md` at the vault root comes first. Skills read it before creating, moving, or renaming notes, and they don't copy its naming or frontmatter schema. Layout: `00_Inbox/`, `10_Daily/`, `20_Work/<project>/` (flat, lowercase), `30_Personal/`, `80_System/Templates/`, `90_Archive/`. Legacy `Work/`, `Personal/`, `Resources/`, and `Attachments/` are read-only. Hub notes are hand-maintained indexes; suggest links to them, don't edit unasked.
- **Wiki-links:** Always use Obsidian `[[Note Name]]` syntax when referencing other vault notes.
- **Templates:** Use existing vault templates when creating new notes. Do not invent new formats.
