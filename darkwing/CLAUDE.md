# darkwing

GitHub code ops. PR creation, structured code review pipeline, and issue management.

## Project Structure

```
skills/          Slash commands and reusable capabilities (SKILL.md files)
.claude-plugin/  Plugin manifest
.mcp.json        MCP server configuration (Qdrant for writing style RAG)
```

## Key Conventions

- **Ghostwriter is central.** All PR reviews, comments, and written output should use the ghostwriter skill for tone and structure.
- **The darkwing skill** (`user-invocable: false`) defines the persona's behavioral patterns. It activates as background context for all GitHub operations.
- **The review skill orchestrates a 4-round pipeline:** gates, design, correctness, verdict. Each round can also be invoked independently.
- **All GitHub/review skills require the `gh` CLI.**
- **MCP:** The `writing-samples` MCP server connects to a local Qdrant instance for writing style RAG. It's optional but improves ghostwriter output quality.

## Skills

### PR Management
| Skill | Purpose |
|-------|---------|
| `/darkwing:pr-status` | Check PR CI, reviews, and merge readiness |
| `/darkwing:pr-create` | Create a PR with structured description |
| `/darkwing:pr-feedback` | Step through unresolved review feedback on your PR |

### Code Review Pipeline
| Skill | Purpose |
|-------|---------|
| `/darkwing:review` | 4-round review orchestrator (gates → design → correctness → verdict) |
| `/darkwing:review-gates` | Round 1: CI/CD status check |
| `/darkwing:review-design` | Round 2: Architecture assessment |
| `/darkwing:review-correctness` | Round 3: Line-by-line code review |
| `/darkwing:review-verdict` | Round 4: Synthesis and approval decision |

### Issue Management
| Skill | Purpose |
|-------|---------|
| `/darkwing:issues` | List, view, create, search, comment, close GitHub issues |
