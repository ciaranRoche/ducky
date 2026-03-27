---
description: GitHub Actions - check workflow status, trigger runs, view logs
allowed-tools: Bash
argument-hint: [action]
disable-model-invocation: true
---

# GitHub Actions

Manage GitHub Actions workflows: check status, view runs, rerun failed, trigger workflows.

## Arguments
- `$1`: Action — `status` (default), `list`, `view [run-id]`, `rerun [run-id]`, `trigger [workflow]`

## Instructions

### Action: status (default)

Show recent workflow runs:

```bash
gh run list --limit 15
```

### Action: list

List available workflows:

```bash
gh workflow list
```

### Action: view [run-id]

Show detailed run info:

```bash
gh run view $2
```

If a run failed, show the failed steps:

```bash
gh run view $2 --log-failed 2>/dev/null | tail -50
```

### Action: rerun [run-id]

Rerun a failed workflow:

```bash
gh run rerun $2 --failed
```

Confirm before rerunning.

### Action: trigger [workflow]

Trigger a workflow dispatch:

```bash
gh workflow run $2
```

Confirm before triggering. Show what workflow will be triggered.

## Output Format

### For status:

```
### GitHub Actions - Recent Runs

| Workflow | Branch | Status | Duration | Triggered |
|----------|--------|--------|----------|-----------|
| CI       | main   | PASS   | 4m 32s   | 2h ago    |
| Deploy   | main   | FAIL   | 12m 5s   | 1h ago    |
| Lint     | feat-x | PASS   | 1m 10s   | 3h ago    |

#### Failed Runs
- **Deploy #456**: Step "integration-tests" failed
  View details: `gh run view 456`
  Rerun failed: `gh run rerun 456 --failed`
```

### For view:

```
### Run #[id]: [workflow-name]

**Status**: PASS/FAIL/IN_PROGRESS
**Branch**: [branch]
**Triggered**: [timestamp] by [actor]
**Duration**: [duration]

#### Jobs
| Job | Status | Duration |
|-----|--------|----------|
| build | PASS | 2m 10s |
| test | FAIL | 5m 30s |

#### Failed Steps (if any)
[Log output from failed steps]
```

### For list:

```
### Available Workflows

| Name | State | File |
|------|-------|------|
| CI | active | .github/workflows/ci.yml |
| Deploy | active | .github/workflows/deploy.yml |
```

## Notes
- For `status`, highlight any currently failing runs prominently
- For `rerun`, only rerun failed jobs by default (not the entire workflow)
- For `trigger`, show the user what they're about to trigger before executing
