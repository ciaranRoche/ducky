# ducky

<p align="center">
  <img src="ducky.png" alt="ducky" width="300">
</p>

Personal AI pair programming toolkit for Claude Code. Four persona-based plugins themed after cartoon ducks, all in your writing style.

## Plugins

| Plugin | Persona | What It Does |
|--------|---------|--------------|
| **ducky** | The rubber duck | Debugging, brainstorming, research, Socratic questioning |
| **scrooge** | Scrooge McDuck | JIRA ticket creation, triage, hygiene, sprint/release reporting, planning |
| **darkwing** | Darkwing Duck | GitHub PRs, structured code review pipeline, issue management |
| **webby** | Webby Vanderquack | Obsidian second brain, daily notes, session logging, vault search |

Install only what you need. Each plugin is self-contained.

## Setup

1. Add the marketplace:
   ```bash
   claude plugin marketplace add github:ciaranRoche/ducky
   ```
2. Install the plugins you want:
   ```bash
   claude plugin install ducky@ducky
   claude plugin install scrooge@ducky
   claude plugin install darkwing@ducky
   claude plugin install webby@ducky
   ```

### Plugin-specific setup

**scrooge** requires [mcp-atlassian](https://github.com/sooperset/mcp-atlassian) and two environment variables:
- `JIRA_USERNAME` — Atlassian email
- `JIRA_API_TOKEN` — Atlassian API token

**darkwing** requires [gh](https://cli.github.com/) (GitHub CLI).

**webby** requires an Obsidian vault. Run `/webby:vault-setup` inside Claude Code, or manually:
```bash
claude mcp add obsidian --scope user -- npx @bitbonsai/mcpvault@latest /path/to/your/vault
```

**Writing style RAG** (optional, all plugins): Start [Qdrant](https://qdrant.tech/) on `localhost:6333` and install [mcp-server-qdrant](https://github.com/qdrant/mcp-server-qdrant). Store writing samples with the `qdrant-store` MCP tool.

## Skills

### ducky (thinking companion)

| Skill | Description |
|-------|-------------|
| `/ducky:duck [topic]` | Rubber duck debugging via Socratic questioning |
| `/ducky:brainstorm [topic]` | Structured brainstorming with thinking frameworks |
| `/ducky:research [topic]` | Technical research with synthesized findings |

### scrooge (JIRA ops)

| Skill | Description |
|-------|-------------|
| `/scrooge:ticket-triage <key>` | Interactive triage — validate, assess, and fix ticket readiness |
| `/scrooge:hygiene <key>\|sprint` | Field completeness and quality checks (single ticket or sprint) |
| `/scrooge:daily-report` | Daily briefing — sprint progress, new tickets, new comments |
| `/scrooge:sprint-report` | Sprint health — burn rate, risks, workload, carry-over |
| `/scrooge:release-status [version]` | Release readiness by Fix Version |
| `/scrooge:sprint-planning` | Sprint planning prep — candidates, stale items, capacity |
| `/scrooge:backlog-grooming` | Backlog audit — staleness, field checks, duplicates |
| `/scrooge:my-sprint` | Current sprint and your assigned tickets |
| `/scrooge:my-tasks` | All your assigned JIRA tasks |
| `/scrooge:story-pointer <key>` | Estimate story points with complexity analysis |
| `/scrooge:set-activity-type <key>` | Set activity type for capacity planning |
| `/scrooge:ticket-creator` | Routes to the correct creator by type |
| `/scrooge:create-bug` | Bug reports with steps to reproduce |
| `/scrooge:create-story` | User stories with acceptance criteria |
| `/scrooge:create-task` | Tech debt, spikes, infrastructure work |
| `/scrooge:create-epic` | Epics with scope and child story breakdown |
| `/scrooge:create-feature` | Portfolio-level initiatives with benefit hypothesis |

### darkwing (GitHub ops)

| Skill | Description |
|-------|-------------|
| `/darkwing:review [PR]` | Multi-pass PR review (orchestrates 4 rounds) |
| `/darkwing:review-gates [PR]` | Round 1: CI/CD status and automated checks |
| `/darkwing:review-design [PR]` | Round 2: Architecture and design assessment |
| `/darkwing:review-correctness [PR]` | Round 3: Line-by-line bugs, security, tests |
| `/darkwing:review-verdict [PR]` | Round 4: Synthesize findings into verdict |
| `/darkwing:pr-create [base]` | Create a PR with structured description |
| `/darkwing:pr-status [PR]` | Check CI, reviews, and merge readiness |
| `/darkwing:pr-feedback [PR]` | Step through open review threads, fixing code or replying |
| `/darkwing:issues [action]` | GitHub issue management |

### webby (second brain)

| Skill | Description |
|-------|-------------|
| `/webby:daily` | Read today's daily note — surface todos and context |
| `/webby:todo <add\|done\|pick>` | Manage todos — add items, mark done, or pick one |
| `/webby:work` | Pick a daily todo and start working on it |
| `/webby:session-log [summary]` | Append session summary to today's daily note |
| `/webby:vault-query <topic>` | Search vault for relevant context |
| `/webby:vault-save <topic>` | File new knowledge into the vault |
| `/webby:vault-setup [path]` | Configure MCPVault for cross-project vault access |
| `/webby:kanban <action>` | Manage kanban board — add, move, done, list, reorder |
| `/webby:kanban-log <task>` | Log activity to a kanban task note |

## Customization

Each plugin includes a **ghostwriter** skill (`skills/ghostwriter/SKILL.md`) with writing style examples and rules. Edit these to match your own voice.
