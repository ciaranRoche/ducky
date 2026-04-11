# scrooge

JIRA project management. Ticket creation, triage, hygiene checks, sprint tracking, and estimation.

## Project Structure

```
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** All ticket descriptions and written output should use the ghostwriter skill for tone and structure.
- **The scrooge skill** (`user-invocable: false`) defines the persona's behavioral patterns. It activates as background context for all JIRA operations.
- **JIRA tickets use wiki markup, not Markdown.** The creator skills have extensive documentation on this. Never use Markdown syntax in JIRA ticket descriptions.
- **Discovering valid JIRA components:** Before setting a component on a ticket, verify it exists: `jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND component is not EMPTY" --plain 2>/dev/null | head -20`. Check that any component you assign is one the project actually uses.
- **JIRA skills require `jira-cli`** (`brew install ankitpokhrel/jira-cli/jira-cli`, then `jira init`). All JIRA skills should check for it before running commands and give a clear setup message if missing.
- **Environment variables:** `DUCKY_JIRA_PROJECT` (default: `HYPERFLEET`) and `JIRA_BASE_URL` (default: `https://redhat.atlassian.net`) configure JIRA integration.
- **MCP:** The `writing-samples` MCP server connects to a local Qdrant instance for writing style RAG. It's optional but improves ghostwriter output quality.
