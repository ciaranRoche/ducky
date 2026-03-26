# ducky

Personal Claude Code plugin. Not a traditional codebase, just markdown configuration files.

## Project Structure

```
agents/          Autonomous agents (AGENT.md files)
commands/        User-invoked slash commands (markdown files)
skills/          Reusable capabilities referenced by commands and agents (SKILL.md files)
.claude/         Claude Code settings
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** Most commands and agents reference the ghostwriter skill for writing style. Any output "on behalf of the user" should go through ghostwriter's tone and structure rules.
- **JIRA tickets use wiki markup, not Markdown.** The ticket creator skill has extensive documentation on this. Never use Markdown syntax in JIRA ticket descriptions.
- **Commands vs Agents:** Commands run inline in the conversation. Agents run autonomously. The `review` command and `reviewer` agent cover the same domain but at different autonomy levels. Same for `research`/`researcher`.
- **Skills are referenced, not invoked directly.** Skills define capabilities that commands and agents activate when relevant.
- **Environment variables:** `DUCKY_JIRA_PROJECT` (default: `HYPERFLEET`) and `JIRA_BASE_URL` (default: `https://issues.redhat.com`) configure JIRA integration.
- **MCP:** The `writing-samples` MCP server connects to a local Qdrant instance for writing style RAG. It's optional but improves ghostwriter output quality.
