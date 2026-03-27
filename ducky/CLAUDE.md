# ducky

Personal Claude Code plugin. Not a traditional codebase, just markdown configuration files.

## Project Structure

```
agents/          Autonomous agent (.md files)
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** Most skills and agents reference the ghostwriter skill for writing style. Any output "on behalf of the user" should go through ghostwriter's tone and structure rules.
- **JIRA tickets use wiki markup, not Markdown.** The ticket creator skill has extensive documentation on this. Never use Markdown syntax in JIRA ticket descriptions.
- **Skills vs Agents:** Skills run inline in the conversation. Agents run autonomously. The `research` skill and `researcher` agent cover the same domain but at different autonomy levels.
- **Some skills are background-only** (ghostwriter, pair-programmer) and set `user-invocable: false`. The rest are user-facing slash commands.
- **Environment variables:** `DUCKY_JIRA_PROJECT` (default: `HYPERFLEET`) and `JIRA_BASE_URL` (default: `https://issues.redhat.com`) configure JIRA integration.
- **MCP:** The `writing-samples` MCP server connects to a local Qdrant instance for writing style RAG. It's optional but improves ghostwriter output quality.
