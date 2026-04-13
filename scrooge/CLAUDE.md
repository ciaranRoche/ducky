# scrooge

JIRA project management. Ticket creation, triage, hygiene checks, sprint tracking, estimation, and nightly report agents.

## Project Structure

```
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (mcp-atlassian for JIRA, Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** All ticket descriptions and written output should use the ghostwriter skill for tone and structure.
- **The scrooge skill** (`user-invocable: false`) defines the persona's behavioral patterns. It activates as background context for all JIRA operations.
- **JIRA tickets use wiki markup, not Markdown.** The creator skills have extensive documentation on this. Never use Markdown syntax in JIRA ticket descriptions.
- **Discovering valid JIRA components:** Before setting a component on a ticket, verify it exists: `jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND component is not EMPTY" --plain 2>/dev/null | head -20`. Check that any component you assign is one the project actually uses.
- **Two JIRA integrations:** Skills use `jira-cli` (Bash). Agents use `mcp-atlassian` (MCP tools). Migration path: once agents prove mcp-atlassian reliability, skills will migrate from jira-cli to MCP.
- **JIRA skills require `jira-cli`** (`brew install ankitpokhrel/jira-cli/jira-cli`, then `jira init`). All JIRA skills should check for it before running commands and give a clear setup message if missing.
- **Environment variables:** `DUCKY_JIRA_PROJECT` (default: `HYPERFLEET`) and `JIRA_BASE_URL` (default: `https://redhat.atlassian.net`) configure JIRA integration. `JIRA_USERNAME` and `JIRA_API_TOKEN` are required for mcp-atlassian (agents).
- **MCP:** Two MCP servers: `atlassian` (mcp-atlassian, read-only JIRA access for agents) and `writing-samples` (Qdrant for writing style RAG, optional).

## MCP-Powered Skills

The following skills use `mcp__atlassian__*` MCP tools (not jira-cli). They are read-only — they surface issues, never modify tickets.

| Skill | Purpose |
|-------|---------|
| `/scrooge:sprint-report` | Sprint health — progress, velocity, blockers, at-risk tickets |
| `/scrooge:new-tickets` | Tickets created in last 24h — grooming quality assessment |
| `/scrooge:backlog-hygiene` | Full backlog audit — field checks, staleness, duplicates |
