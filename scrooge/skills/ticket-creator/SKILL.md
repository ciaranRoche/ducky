---
name: ticket-creator
description: Routes to the correct JIRA ticket creator based on type. Activates when users ask to "create a ticket" or "file a ticket" without specifying a type (bug, story, task, epic, or feature).
---

# JIRA Ticket Creator (Router)

This skill routes to the appropriate type-specific creator. If the user's request clearly indicates a type, delegate directly. Otherwise, ask.

## When to Use This Skill

Activate when the user says something generic like:
- "create a ticket", "file a ticket", "I need a JIRA ticket"
- "can you create a ticket for...?"
- "add a ticket to the backlog"

Do NOT activate if the user specifies a type -- let the type-specific skill handle it directly.

## Routing Logic

1. If the request mentions **bug**, **defect**, **broken**, **regression**, or **not working**: invoke **create-bug**
2. If the request mentions **story** or **user story**: invoke **create-story**
3. If the request mentions **task**, **spike**, **tech debt**, **investigate**, or **infrastructure**: invoke **create-task**
4. If the request mentions **epic**: invoke **create-epic**
5. If the request mentions **feature** or **initiative**: invoke **create-feature**
6. If unclear, ask:

> What type of ticket would you like to create?
>
> - **Bug** -- something is broken, needs steps to reproduce
> - **Story** -- user-facing work with acceptance criteria
> - **Task** -- tech debt, infrastructure, documentation, or spike
> - **Epic** -- large body of work containing multiple stories
> - **Feature** -- portfolio-level initiative containing multiple epics

Then invoke the corresponding skill.

## Prerequisites

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`

## Available Creator Skills

| Type | Skill | Use For |
|------|-------|---------|
| Bug | `create-bug` | Defects with steps to reproduce, expected/actual behavior |
| Story | `create-story` | User stories with AC and technical notes |
| Task | `create-task` | Tech debt, infrastructure, documentation, spikes |
| Epic | `create-epic` | Scoped body of work with child stories |
| Feature | `create-feature` | Portfolio-level initiative with benefit hypothesis and child epics |
