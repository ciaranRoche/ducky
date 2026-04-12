---
name: vault-save
description: File new knowledge into the Obsidian vault at the correct location with
  proper frontmatter and wiki-links.
allowed-tools:
  - mcp__obsidian__write_note
  - mcp__obsidian__search_notes
  - mcp__obsidian__read_note
  - mcp__obsidian__patch_note
  - mcp__obsidian__update_frontmatter
argument-hint: '<topic or content>'
---

# Vault Save: File New Knowledge

Create a new note in the Obsidian vault at the correct location, with proper frontmatter, wiki-links to related notes, and content written in the user's voice.

## Behavior

1. **Determine what to save** from the argument or conversation context. Ask the user to clarify if unclear.
2. **Choose the correct vault location** based on content type:

   | Content Type | Location | Template Reference |
   |---|---|---|
   | Work technical learning | `Work/Knowledge/` | Knowledge-Template |
   | Personal technical learning | `Personal/Knowledge/` | Knowledge-Template |
   | Work project documentation | `Work/Projects/[Project]/` | Work-Project-Template |
   | Personal project documentation | `Personal/Projects/` | Project-Template |
   | Meeting notes | `Work/Meetings/` | Meeting-Template |
   | Technical design/decision | `Work/Projects/[Project]/` | Technical-Doc-Template |
   | Homelab/infrastructure | `Personal/Projects/` | Project-Template |

   If the location is ambiguous, ask the user.

3. **Search for related notes** using `mcp__obsidian__search_notes` with relevant terms. Identify 2-5 related notes to link to.
4. **Create the note** using `mcp__obsidian__write_note`:
   - Use descriptive file names with spaces (Obsidian-friendly)
   - Include frontmatter with at minimum: `date`, `type`, `tags`
   - Add wiki-links to related notes in the body
   - Apply the ghostwriter skill for writing tone
   - Structure content with clear headings
5. **Report what was created**:
   - File path
   - Why that location was chosen
   - Tags applied
   - Related notes linked
   - Suggest hub notes to link from if appropriate (`[[Work Projects Hub]]`, `[[Home Lab Hub]]`, etc.)

6. **Optionally update today's daily note** `#### Related Notes` section to reference the new note. Only do this if the user is actively working in a daily note context (e.g., they ran `/daily` earlier in the session). Use `mcp__obsidian__patch_note` to add the wiki-link.

## Frontmatter Conventions

```yaml
---
date: YYYY-MM-DD
type: knowledge | project | meeting | technical-doc
tags:
  - relevant-tag
---
```

## Notes

- Never create notes in the vault root. Always use the appropriate subdirectory.
- Use the existing directory structure. Do not create new top-level directories.
- File names should be descriptive and use spaces: `Kubernetes Pod Security Standards.md`, not `k8s-pod-security.md`
- When linking to hub notes, use the exact wiki-link names: `[[Work Projects Hub]]`, `[[Quarterly Hub]]`, `[[Home Lab Hub]]`
- If a note on this topic already exists, tell the user and offer to update it instead of creating a duplicate
