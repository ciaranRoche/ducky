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
- **MCP:** The `atlassian` MCP server (mcp-atlassian) provides JIRA access for all skills. The `writing-samples` server (provided by the `ducky` plugin) connects to Qdrant for writing style RAG (optional).

## Custom Field Reference

| Field ID | Name | Type | Usage |
|----------|------|------|-------|
| `customfield_10016` | Story point estimate | Number | Primary story points field (next-gen) — check this first |
| `customfield_10028` | Story Points | Number | Classic field — fallback if 10016 is empty |
| `customfield_10464` | Activity Type | Select | Set as `{"value": "Type Name"}` in `additional_fields` |
| `customfield_10011` | Epic Name | String | Required when creating epics |
| `customfield_10014` | Epic Link | Any | Links issues to parent epics |

## Fix Version

Fix Version (`fixVersions`) is a standard JIRA field — not a custom field. It ties issues to a specific release. All hygiene and audit skills surface missing Fix Versions as an informational highlight (not a required field gate).

- **Querying:** Include `fixVersions` in the `fields` parameter when fetching issues
- **Setting:** `fields: {"fixVersions": [{"name": "Version Name"}]}`
- **Validating:** Use `mcp__atlassian__jira_get_project_versions` with `project_key: HYPERFLEET` to get available versions
- **Release planning:** Use `/scrooge:release-status` to assess readiness of a specific Fix Version

## Discovering Valid Components

Use `mcp__atlassian__jira_get_project_components` with `project_key: HYPERFLEET` to get the list of valid components. Check that any component you assign is one the project actually uses.

## Skills

All skills use `mcp__atlassian__jira_*` MCP tools.

### Triage & Hygiene
| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `/scrooge:ticket-triage <key>` | Interactive triage with validation | Deep-dive a single ticket before sprint |
| `/scrooge:hygiene <key>\|sprint` | Field completeness and quality checks | Quick validation of one ticket or bulk sprint audit |

### Reports
| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `/scrooge:daily-report` | Sprint progress + new tickets + new comments | Start of day orientation |
| `/scrooge:sprint-report` | Full sprint health — burn rate, risks, workload | Standups, mid-sprint check-ins |
| `/scrooge:release-status [version]` | Release readiness by Fix Version | Release planning, status updates |

### Planning & Grooming
| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `/scrooge:sprint-planning` | Backlog candidates, stale items, capacity analysis | Before sprint planning meetings |
| `/scrooge:backlog-grooming` | Full backlog audit — staleness, duplicates, field gaps | Weekly grooming sessions |

### Personal Views
| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `/scrooge:my-sprint` | Your assigned tickets in the active sprint | Quick personal check |
| `/scrooge:my-tasks` | All your assigned tickets across project | Full workload view |

### Estimation & Fields
| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `/scrooge:story-pointer <key>` | Estimate story points with historical comparison | Pointing tickets during grooming |
| `/scrooge:set-activity-type <key>` | Set activity type for capacity planning | Fill missing activity types |

### Create
| Skill | Purpose |
|-------|---------|
| `/scrooge:ticket-creator` | Router — delegates to type-specific creator |
| `/scrooge:create-bug` | Create bug with steps to reproduce |
| `/scrooge:create-story` | Create story with acceptance criteria |
| `/scrooge:create-task` | Create task, spike, or tech debt ticket |
| `/scrooge:create-epic` | Create epic with scope and success criteria |
| `/scrooge:create-feature` | Create portfolio-level feature |
