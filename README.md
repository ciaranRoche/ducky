# ducky

Personal AI pair programming toolkit for Claude Code. Three persona-based plugins themed after cartoon ducks, all in your writing style.

## Plugins

| Plugin | Persona | What It Does |
|--------|---------|--------------|
| **ducky** | The rubber duck | Debugging, brainstorming, research, Socratic questioning |
| **scrooge** | Scrooge McDuck | JIRA ticket creation, triage, sprint tracking, estimation |
| **darkwing** | Darkwing Duck | GitHub PRs, structured code review pipeline, issue management |

Install only what you need. Each plugin is self-contained.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) CLI
- [gh](https://cli.github.com/) (GitHub CLI) — for darkwing
- [jira-cli](https://github.com/ankitpokhrel/jira-cli) (`jira init`) — for scrooge
- [Qdrant](https://qdrant.tech/) on `localhost:6333` (optional, for writing style RAG)
- [mcp-server-qdrant](https://github.com/qdrant/mcp-server-qdrant) (optional, MCP bridge)

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
   ```
3. Set environment variables (for scrooge):
   - `DUCKY_JIRA_PROJECT` — JIRA project key (default: `HYPERFLEET`)
   - `JIRA_BASE_URL` — JIRA instance URL (default: `https://issues.redhat.com`)

If using writing style RAG:
1. Start Qdrant: `docker run -p 6333:6333 qdrant/qdrant`
2. Store writing samples with the `qdrant-store` MCP tool

## Skills

### ducky (thinking companion)

| Skill | Description |
|-------|-------------|
| `/ducky:duck [topic]` | Rubber duck debugging via Socratic questioning |
| `/ducky:brainstorm [topic]` | Structured brainstorming with thinking frameworks |
| `/ducky:research [topic]` | Technical research with synthesized findings |

Background: **ghostwriter** (writing style), **pair-programmer** (Socratic questioning engine)
Agent: **researcher** (autonomous deep-dive research)

### scrooge (JIRA ops)

| Skill | Description |
|-------|-------------|
| `/scrooge:create-story` | User stories with acceptance criteria |
| `/scrooge:create-bug` | Bug reports with steps to reproduce |
| `/scrooge:create-task` | Tech debt, spikes, infrastructure work |
| `/scrooge:create-epic` | Epics with child story breakdown |
| `/scrooge:create-feature` | Portfolio-level initiatives with benefit hypothesis |
| `/scrooge:ticket-creator` | Routes to the correct creator by type |
| `/scrooge:story-pointer` | Estimate story points with complexity analysis |
| `/scrooge:ticket-triage` | Validate a ticket's sprint readiness |
| `/scrooge:triage [scope]` | Bulk audit sprint tickets for completeness |
| `/scrooge:my-sprint` | Current sprint and your assigned tickets |
| `/scrooge:my-tasks` | All your assigned JIRA tasks |
| `/scrooge:new-comments` | Tickets with new comments you may have missed |
| `/scrooge:sprint-status` | Sprint health overview for team leads |

Background: **ghostwriter** (writing style), **scrooge** (persona behavioral patterns)

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
| `/darkwing:issues [action]` | GitHub issue management |

Background: **ghostwriter** (writing style), **darkwing** (persona behavioral patterns)

## Customization

The ghostwriter skill (in each plugin's `skills/ghostwriter/SKILL.md`) contains writing style examples and rules. Edit these to match your own voice, tone, and formatting preferences.

For RAG-based style matching, store samples in Qdrant using the `qdrant-store` MCP tool.
