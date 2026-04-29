---
name: standup
description: Summarize your last session log into a standup-ready recap, grouped by
  project. Run this before standup to get oriented on what you did.
allowed-tools:
  - mcp__obsidian__read_note
  - mcp__obsidian__list_directory
  - Bash
argument-hint: '[YYYY-MM-DD]'
---

# Standup: Yesterday's Log Summary

Summarize the most recent session log into a concise, project-grouped standup recap.

## Behavior

### 1. Resolve the target date

- If the user provides a date argument (e.g., `/standup 2026-04-25`), use that as the target date. Validate it matches `YYYY-MM-DD` format.
- If no argument, compute yesterday's date using `date -d 'yesterday' +%Y-%m-%d` via Bash.
- Also compute today's date using `date +%Y-%m-%d` (needed for fallback comparison).
- If the target date is in the future, tell the user and stop.

### 2. Read the target daily note

- Read the daily note at `Daily/<target-date>.md` using `mcp__obsidian__read_note`.
- Extract the `#### Log` section: content between `#### Log` and the next `####` heading (typically `#### A day in review`).

### 3. Fallback if the note is missing or the log is empty

A log is considered empty if it does not exist or contains only the placeholder `- `.

- List the `Daily/` directory using `mcp__obsidian__list_directory`.
- Filter to `.md` files only. Ignore sync-conflict files (`.sync-conflict-*.md`).
- Sort filenames descending (they are `YYYY-MM-DD.md` so lexicographic sort works).
- Find the first file whose date is strictly before today.
- Read that note and extract its `#### Log` section.
- If that log is also empty, report "No recent log entries found" and stop.
- Note the actual date used so the output header reflects the real date.

### 4. Group entries by project

- Parse each log line. Lines matching `- [tag] description` are grouped under that tag.
- Lines without a `[tag]` prefix go into a "General" group.
- Strip the leading `- ` and `[tag] ` to get clean descriptions.

### 5. Summarize

- If a project has **5 or fewer entries**, present them as-is.
- If a project has **more than 5 entries**, consolidate related items into high-level themes. Preserve ticket/PR references that represent discrete deliverables.
- Drop process noise that adds no standup value (e.g., "ran daily report", "checked sprint board").
- Target **10-20 lines of output total** across all projects.

### 6. Format the output

If the log date matches yesterday:

```
### Standup

**Yesterday** (YYYY-MM-DD):
_[project-a]_
- Summary item 1
- Summary item 2

_[project-b]_
- Summary item 3
```

If the log date differs from yesterday (e.g., Friday's log on Monday):

```
### Standup

**Last session** (YYYY-MM-DD, DayOfWeek):
_[project-a]_
- Summary item 1
- Summary item 2
```

Use `date -d 'YYYY-MM-DD' +%A` to compute the day-of-week label for non-yesterday dates.

If all entries belong to a single project, skip the project subheading and just list the items directly under the date header.

## Edge Cases

- **Monday morning:** Fallback walks past Saturday/Sunday to Friday automatically.
- **Vacation/extended gap:** Fallback handles arbitrary gaps since it lists the full directory.
- **Future date argument:** Warn the user, do not attempt to read a future note.
- **No `[tag]` prefix on any entries:** Group everything under "General" but omit the subheading (same as single-project behavior).
- **Empty vault / no daily notes at all:** Report "No daily notes found in the vault" and stop.

## Notes

- Apply the ghostwriter skill for tone
- This is a read-only skill. It never modifies vault content.
- Keep the output brief and scannable. This is a communication artifact, not a journal entry.
