# webby

Obsidian second brain. Daily notes, session logging, vault search, and knowledge capture.

## Project Structure

```
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
```

## Key Conventions

- **Ghostwriter is central.** All vault entries and written output should use the ghostwriter skill for tone and structure.
- **The webby skill** (`user-invocable: false`) defines the persona's behavioral patterns. It activates as background context for all vault operations.
- **MCP:** The `obsidian` MCP server is configured at user scope (run `/vault-setup` if missing). The `writing-samples` server (provided by the `ducky` plugin) connects to Qdrant for writing style RAG (optional).
- **Daily notes** live at `Daily/YYYY-MM-DD.md` and follow a consistent template with sections for todos, log, review, and related notes.
- **Kanban board** lives at `Work/Kanban.md` (Obsidian Kanban plugin format). Task notes are stored in the folder specified by the board's `new-note-folder` setting (default: `Work/Tasks/`). Task notes include a `#### Log` section for per-task activity tracking.
- **Vault structure:** Work vs Personal separation. Knowledge, Projects, Meetings directories. Hub notes serve as indexes.
- **Wiki-links:** Always use Obsidian `[[Note Name]]` syntax when referencing other vault notes.
- **Templates:** Use existing vault templates when creating new notes. Do not invent new formats.
