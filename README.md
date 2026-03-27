# ducky

Personal AI pair programming toolkit for Claude Code. Rubber duck debugging, PR reviews, brainstorming, research, JIRA ops, and GitHub ops, all in your writing style.

## What This Is

A Claude Code plugin that bundles:

- **Commands** (`/review`, `/research`, `/brainstorm`, etc.) for quick interactive workflows
- **Skills** (ghostwriter, pair-programmer, JIRA ticket creator, etc.) for reusable capabilities
- **Agents** (researcher) for autonomous multi-step tasks
- **MCP integration** with Qdrant for writing style RAG

The ghostwriter skill is the backbone. It defines a personal writing style that other commands and agents reference, so all generated output sounds like you.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) CLI
- [gh](https://cli.github.com/) (GitHub CLI, authenticated)
- [jira-cli](https://github.com/ankitpokhrel/jira-cli) (configured with `jira init`)
- [Qdrant](https://qdrant.tech/) running on `localhost:6333` (optional, for writing style RAG)
- [mcp-server-qdrant](https://github.com/qdrant/mcp-server-qdrant) (optional, MCP bridge to Qdrant)

## Setup

1. Add the marketplace:
   ```bash
   claude plugin marketplace add github:ciaranRoche/ducky
   ```
2. Install the plugin:
   ```bash
   claude plugin install ducky@ducky
   ```
3. Set environment variables (optional):
   - `DUCKY_JIRA_PROJECT` - JIRA project key (default: `HYPERFLEET`)
   - `JIRA_BASE_URL` - JIRA instance URL (default: `https://issues.redhat.com`)

If using the writing style RAG:
1. Start Qdrant: `docker run -p 6333:6333 qdrant/qdrant`
2. The `mcp-server-qdrant` server is configured in `.mcp.json`
3. Store writing samples with the `qdrant-store` tool to build your style corpus

## Commands

| Command | Description |
|---------|-------------|
| `/duck [topic]` | Rubber duck debugging via Socratic questioning |
| `/review [PR]` | Review a PR with feedback in your writing style |
| `/pr-create [base]` | Create a PR with a well-structured description |
| `/pr-status [PR]` | Check CI, reviews, and merge readiness |
| `/research [topic]` | Deep research with synthesized findings |
| `/brainstorm [topic]` | Structured brainstorming with thinking frameworks |
| `/issues [action]` | GitHub issue management |
| `/gh-actions [action]` | GitHub Actions workflow management |
| `/my-sprint` | Current sprint and your assigned tickets |
| `/my-tasks` | All your assigned JIRA tasks |
| `/new-comments` | Tickets with new comments you may have missed |
| `/sprint-status` | Sprint health overview for team leads |
| `/triage [scope]` | Audit sprint tickets for completeness |

## Skills

| Skill | Description |
|-------|-------------|
| **ghostwriter** | Personal writing style applied to all output |
| **pair-programmer** | Rubber duck debugging via Socratic questioning |
| **jira-ticket-creator** | Create well-structured JIRA tickets via CLI |
| **jira-story-pointer** | Estimate story points with complexity analysis |
| **jira-triage** | Validate ticket quality for sprint readiness |

## Agents

| Agent | Description |
|-------|-------------|
| **researcher** | Autonomous multi-source technical research |

## Customization

The ghostwriter skill (`skills/ghostwriter/SKILL.md`) contains writing style examples and rules. Edit these to match your own voice, tone, and formatting preferences.

For RAG-based style matching, store samples in Qdrant using the `qdrant-store` MCP tool. This gives better results than static examples and improves over time as you add more samples.
