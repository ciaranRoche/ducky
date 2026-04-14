# scrooge

JIRA project management. Ticket creation, triage, hygiene checks, sprint tracking, estimation, and reporting.

## Project Structure

```
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (mcp-atlassian for JIRA, Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** All ticket descriptions and written output should use the ghostwriter skill for tone and structure.
- **The scrooge skill** (`user-invocable: false`) defines the persona's behavioral patterns. It activates as background context for all JIRA operations.
- **Descriptions use Markdown.** The MCP server converts to JIRA's native format (ADF) automatically. Do not use JIRA wiki markup — write in standard Markdown.
- **No curly braces in content.** `{}` breaks JIRA rendering (learned from HYPERFLEET-258). Use `:id` or SCREAMING_CASE for path parameters.
- **All skills use mcp-atlassian MCP tools** (`mcp__atlassian__jira_*`). No jira-cli dependency.
- **Environment variables:** `JIRA_USERNAME` and `JIRA_API_TOKEN` are required for mcp-atlassian authentication (Atlassian Cloud API token auth).
- **MCP:** Two MCP servers: `atlassian` (mcp-atlassian, JIRA access for all skills) and `writing-samples` (Qdrant for writing style RAG, optional).

## Custom Field Reference

| Field ID | Name | Type | Usage |
|----------|------|------|-------|
| `customfield_10016` | Story point estimate | Number | Primary story points field (next-gen) — check this first |
| `customfield_10028` | Story Points | Number | Classic field — fallback if 10016 is empty |
| `customfield_10464` | Activity Type | Select | Set as `{"value": "Type Name"}` in `additional_fields` |
| `customfield_10011` | Epic Name | String | Required when creating epics |
| `customfield_10014` | Epic Link | Any | Links issues to parent epics |

## Discovering Valid Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to get the list of valid components. Check that any component you assign is one the project actually uses.

## Skills

All skills use `mcp__atlassian__jira_*` MCP tools.

### Read-Only Skills
| Skill | Purpose |
|-------|---------|
| `/scrooge:my-sprint` | Current sprint and your assigned tasks |
| `/scrooge:my-tasks` | All your assigned tickets |
| `/scrooge:sprint-status` | Sprint health overview for team leads |
| `/scrooge:sprint-report` | Sprint health — progress, velocity, blockers, at-risk tickets |
| `/scrooge:new-tickets` | Tickets created in last 24h — grooming quality assessment |
| `/scrooge:new-comments` | Tickets with recent comments you may have missed |
| `/scrooge:backlog-hygiene` | Full backlog audit — field checks, staleness, duplicates |
| `/scrooge:ticket-hygiene` | Validate required fields on a specific ticket |
| `/scrooge:sprint-hygiene` | Bulk sprint audit — fields, components, duplicates |

### Read+Write Skills
| Skill | Purpose |
|-------|---------|
| `/scrooge:story-pointer` | Estimate story points with historical comparison |
| `/scrooge:set-activity-type` | Set activity type for capacity planning |
| `/scrooge:ticket-triage` | Interactive triage — assess and fix ticket readiness |

### Create Skills
| Skill | Purpose |
|-------|---------|
| `/scrooge:ticket-creator` | Router — delegates to type-specific creator |
| `/scrooge:create-bug` | Create bug with steps to reproduce |
| `/scrooge:create-story` | Create story with acceptance criteria |
| `/scrooge:create-task` | Create task, spike, or tech debt ticket |
| `/scrooge:create-epic` | Create epic with scope and success criteria |
| `/scrooge:create-feature` | Create portfolio-level feature |
