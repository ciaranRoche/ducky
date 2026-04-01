# ducky

Rubber duck thinking companion. Debugging, brainstorming, research, and Socratic questioning.

## Project Structure

```
agents/          Autonomous agent (.md files)
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** Most skills and agents reference the ghostwriter skill for writing style. Any output "on behalf of the user" should go through ghostwriter's tone and structure rules.
- **Skills vs Agents:** Skills run inline in the conversation. Agents run autonomously. The `research` skill is for quick inline lookups (few searches, fast answer). The `researcher` agent is for deep multi-phase investigations that need cross-referencing and comprehensive synthesis.
- **Some skills are background-only** (ghostwriter, pair-programmer) and set `user-invocable: false`. The rest are user-facing slash commands.
- **MCP:** The `writing-samples` MCP server connects to a local Qdrant instance for writing style RAG. It's optional but improves ghostwriter output quality.
