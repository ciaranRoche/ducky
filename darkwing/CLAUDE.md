# darkwing

GitHub code ops. PR creation, structured code review pipeline, and issue management.

## Project Structure

```
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** All PR reviews, comments, and written output should use the ghostwriter skill for tone and structure.
- **The darkwing skill** (`user-invocable: false`) defines the persona's behavioral patterns. It activates as background context for all GitHub operations.
- **The review skill orchestrates a 4-round pipeline:** gates, design, correctness, verdict. Each round can also be invoked independently.
- **All GitHub/review skills require the `gh` CLI.**
- **MCP:** The `writing-samples` MCP server connects to a local Qdrant instance for writing style RAG. It's optional but improves ghostwriter output quality.
