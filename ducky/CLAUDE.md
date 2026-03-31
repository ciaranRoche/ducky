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

- **Ghostwriter is central.** Most skills and agents reference the ducky-ghostwriter skill for writing style. Any output "on behalf of the user" should go through ghostwriter's tone and structure rules.
- **JIRA tickets use wiki markup, not Markdown.** The ticket creator skill has extensive documentation on this. Never use Markdown syntax in JIRA ticket descriptions.
- **Discovering valid JIRA components:** Before setting a component on a ticket, verify it exists: `jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND component is not EMPTY" --plain 2>/dev/null | head -20`. Check that any component you assign is one the project actually uses.
- **Skills vs Agents:** Skills run inline in the conversation. Agents run autonomously. The `ducky-research` skill is for quick inline lookups (few searches, fast answer). The `researcher` agent is for deep multi-phase investigations that need cross-referencing and comprehensive synthesis.
- **Some skills are background-only** (ducky-ghostwriter, ducky-pair-programmer) and set `user-invocable: false`. The rest are user-facing slash commands.
- **JIRA skills require `jira-cli`** (`brew install ankitpokhrel/jira-cli/jira-cli`, then `jira init`). All JIRA skills should check for it before running commands and give a clear setup message if missing.
- **Environment variables:** `DUCKY_JIRA_PROJECT` (default: `HYPERFLEET`) and `JIRA_BASE_URL` (default: `https://issues.redhat.com`) configure JIRA integration.
- **MCP:** The `writing-samples` MCP server connects to a local Qdrant instance for writing style RAG. It's optional but improves ghostwriter output quality.
