---
name: vault-query
description: Search the Obsidian vault for relevant context on a topic. Returns synthesized
  findings with wiki-links.
allowed-tools:
  - mcp__obsidian__search_notes
  - mcp__obsidian__read_note
  - mcp__obsidian__read_multiple_notes
argument-hint: '<topic>'
---

# Vault Query: Search the Knowledge Base

Search the Obsidian vault for notes relevant to a topic and synthesize the findings.

## Behavior

1. **Take the user's query** from the argument
2. **Search the vault** using `mcp__obsidian__search_notes` with `searchContent: true` and a reasonable limit (5-10 results)
3. **Read the top results** using `mcp__obsidian__read_note` or `mcp__obsidian__read_multiple_notes` (batch, max 10) to get full content
4. **Synthesize findings**:
   - Group results by vault section (Work/Knowledge, Personal/Projects, Daily, etc.)
   - Summarize the relevant content from each note
   - Use `[[Note Name]]` wiki-links for all referenced notes
   - Highlight connections between notes if any exist
   - Apply the ghostwriter skill for tone

### Output Format

```
### Vault: [query]

Found X relevant notes.

**[[Note Title]]** (Work/Knowledge/)
Brief summary of the relevant content from this note.

**[[Note Title]]** (Personal/Projects/)
Brief summary of the relevant content from this note.

**Connections:**
- [[Note A]] references the same pattern discussed in [[Note B]]
```

If no results are found, say so and suggest alternative search terms or vault sections the user might check manually.

## Notes

- Do not dump raw note content. Summarize what's relevant to the query.
- If a note is very long, focus on the sections most relevant to the query
- Include the vault directory path for each result so the user knows where it lives
- Keep the synthesis concise. The goal is to surface relevant context, not reproduce the vault
