---
name: jira-sprint-status
description: Sprint health overview for team leads - progress, blockers, and risks
allowed-tools: Bash
argument-hint: [project-key]
---

# Sprint Status (Team Lead View)

Comprehensive sprint health report for team leads and scrum masters.

## Arguments
- `$1` (optional): Project key. Defaults to `$DUCKY_JIRA_PROJECT` (or `HYPERFLEET` if unset).

## Instructions

1. **Get current sprint info:**
   ```bash
   jira sprint list --current -p ${DUCKY_JIRA_PROJECT:-HYPERFLEET} --plain 2>/dev/null
   ```

2. **Get all tickets in current sprint (with full details):**
   ```bash
   jira sprint list --current -p ${DUCKY_JIRA_PROJECT:-HYPERFLEET} --raw 2>/dev/null
   ```

3. **Get tickets by status - To Do:**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND status = 'To Do' AND sprint in openSprints()" --plain 2>/dev/null
   ```

4. **Get tickets by status - In Progress:**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND status = 'In Progress' AND sprint in openSprints()" --plain 2>/dev/null
   ```

5. **Get tickets by status - Done (this sprint):**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND status = Done AND sprint in openSprints()" --plain 2>/dev/null
   ```

6. **Find blockers (high priority not done):**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND priority = Highest AND status != Done" --plain 2>/dev/null
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND priority = High AND status != Done" --plain 2>/dev/null
   ```

7. **Find unassigned tickets:**
   ```bash
   jira issue list -q"project = ${DUCKY_JIRA_PROJECT:-HYPERFLEET} AND assignee is EMPTY AND sprint in openSprints()" --plain 2>/dev/null
   ```

## Output Format

### Sprint Overview
```
Sprint: [Sprint Name]
Duration: [Start Date] - [End Date]
Days Remaining: X days
```

### Progress Summary
| Status | Count | Story Points |
|--------|-------|--------------|
| To Do | X | X pts |
| In Progress | X | X pts |
| Done | X | X pts |
| **Total** | **X** | **X pts** |

Progress: [=========>    ] 65% complete

### Risk Assessment

#### Blockers & High Priority
- TICKET-1: [Summary] - Assigned to [Name] - [X days in status]
- TICKET-2: [Summary] - **UNASSIGNED**

#### At-Risk Items
- Tickets in progress > 5 days without update
- Tickets without story points
- Unassigned tickets

#### Carry-Over Risk
- Tickets likely to spill: X
- Story points at risk: X

### Team Workload
| Team Member | To Do | In Progress | Done |
|-------------|-------|-------------|------|
| [Name] | X | X | X |

### Recommendations
- List actionable items for the team lead
- Highlight tickets needing attention
- Suggest re-assignments if workload is unbalanced

If jira-cli is not installed or configured, inform the user they need to:
1. Install jira-cli: `brew install ankitpokhrel/jira-cli/jira-cli`
2. Configure it: `jira init`
