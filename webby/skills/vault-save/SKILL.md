---
name: vault-save
description: File new knowledge into the Obsidian vault at the correct location with
  proper frontmatter and wiki-links, and link it from today's daily note.
allowed-tools:
  - mcp__obsidian__write_note
  - mcp__obsidian__search_notes
  - mcp__obsidian__read_note
  - mcp__obsidian__get_note_outline
  - mcp__obsidian__patch_note
  - mcp__obsidian__update_frontmatter
  - mcp__obsidian__list_directory
  - Bash
argument-hint: '<topic or content>'
---

# Vault Save: File New Knowledge

Create a new note in the Obsidian vault at the correct location, with frontmatter, wiki-links to related notes, content in the user's voice, and a link back from today's daily note.

## Behavior

1. **Read `_AGENT_MANIFEST.md`** at the vault root with `mcp__obsidian__read_note`. It is the source of truth for folders, file naming, frontmatter, and wiki-link format. If anything below conflicts with it, the manifest wins.
2. **Determine what to save** from the argument or conversation context. Ask if unclear.
3. **Choose the location** by content type:

   | Content Type | Location | Template | Daily bullet |
   |---|---|---|---|
   | Work technical learning | `20_Work/knowledge/` | Knowledge-Template | Learning |
   | Personal technical learning | `30_Personal/knowledge/` | Knowledge-Template | Learning |
   | Work project documentation | `20_Work/<project>/` | Work-Project-Template | Projects |
   | Technical design/decision | `20_Work/<project>/` | Technical-Doc-Template | Projects |
   | Personal project documentation | `30_Personal/projects/` | Project-Template | Projects |
   | Homelab/infrastructure | `30_Personal/projects/` | Project-Template | Projects |
   | Meeting notes | `20_Work/meetings/` | Meeting-Template | Meetings |
   | Project unknown / unsorted | `00_Inbox/` | closest match | Learning |

   Templates live in `80_System/Templates/`. `20_Work/<project>/` is flat and lowercase (e.g. `20_Work/hyperfleet/`): list `20_Work/` with `mcp__obsidian__list_directory` to pick the project folder, and never create subfolders inside it. Never write to the legacy `Work/`, `Personal/`, `Resources/`, or `Attachments/` folders. If the location is ambiguous, ask.

4. **Check for an existing note** on the topic with `mcp__obsidian__search_notes`. If one exists, offer to update it instead of duplicating.
5. **Find 2-5 related notes** to link to (same search).
6. **Create the note** with `mcp__obsidian__write_note`:
   - File name per the manifest: `[ID]-[project]-[slug].md`, where `ID` is `date +%Y%m%d%H%M%S` taken at creation (e.g. `20260917143000-hyperfleet-pod-security-standards.md`)
   - Frontmatter per the manifest schema (see below). Use the template's headings for the body structure.
   - Wiki-links to related notes in the body, in the manifest format `[[full-filename|Title]]`
   - Ghostwriter skill for tone
7. **Link it from today's daily note.** This is required, not optional:
   - `date +%Y-%m-%d`, read `10_Daily/YYYY-MM-DD.md`. If it does not exist, skip this step and tell the user (do not create the daily note from here).
   - Confirm `#### Related Notes` exists via `mcp__obsidian__get_note_outline`; if missing, tell the user and skip.
   - Patch the matching bullet (`- Projects: `, `- Meetings: `, `- Learning: `) to append `[[full-filename|Title]]`, comma-separated if links already exist.
   - Example: `oldString` `- Learning: ` -> `newString` `- Learning: [[20260917143000-hyperfleet-pod-security-standards|Pod Security Standards]]`
8. **Report**: path, why that location, tags, related notes linked, and which hub note (`[[Work Projects Hub]]`, `[[Home Lab Hub]]`, `[[Technology Hub]]`) should get a link if it is a project or knowledge note. Hubs are hand-maintained, offer to patch the hub, do not do it unasked.

## Frontmatter Conventions

Follow the schema in `_AGENT_MANIFEST.md` exactly: the keys `id`, `type`, `status`, `project`, `tags`, `aliases`, in that order. `id` is quoted and equals the filename ID. `tags` and `aliases` are always arrays, and the first alias is the human title. Do not copy the schema from here; re-read the manifest so changes to it are picked up.

Extra keys go after the required ones, e.g. `jira:` (a list of ticket keys, used instead of ticket keys as tags).

Tag rules: lowercase, hyphenated, singular (`adapter` not `adapters`), reuse an existing tag over inventing a synonym (`multitenancy` not `multi-tenancy`, `postgres` not `postgresql`). Ticket keys and PR numbers go in `jira:` or the body, never as tags.

## Notes

- Never create notes in the vault root (only `_AGENT_MANIFEST.md` and `_ACTIVE_WORKSPACE.md` live there). Never create new top-level directories.
- Use the existing directory structure and templates. Do not invent new formats.
