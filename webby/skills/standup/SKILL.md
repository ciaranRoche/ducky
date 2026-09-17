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

# Standup: Last Session's Log Summary

Summarize the most recent session log into a concise, project-grouped standup recap.

## Behavior

### 1. Resolve the target date

- Compute today with `date +%Y-%m-%d`.
- If the user gave a date argument, validate `YYYY-MM-DD`, refuse future dates, and use it directly.
- Otherwise find the most recent daily note before today:
  - List `10_Daily/` with `mcp__obsidian__list_directory`
  - Keep `.md` files only, drop `.sync-conflict-*.md`
  - Sort descending (filenames are `YYYY-MM-DD.md`, lexicographic order is date order)
  - Take the first file whose date is strictly before today
  - This handles weekends, holidays, and gaps without any `date -d` arithmetic

### 2. Read the log

- Read `10_Daily/<target-date>.md` with `mcp__obsidian__read_note`
- Extract `#### Log`: content between `#### Log` and the next `####` heading
- A log is empty if the section is missing or contains only `- `. If empty, continue down the sorted list to the next earlier note (max 5 hops). If still nothing, report "No recent log entries found" and stop.
- Note the date actually used so the header is honest.

### 3. Group entries by project

- Lines matching `- [tag] description` group under that tag
- Lines without a prefix go under "General"
- Strip the leading `- ` and `[tag] ` to get clean descriptions
- Drop `- Started: ...` lines (they are work-in-progress markers, not outcomes) unless there is nothing else

### 4. Summarize

- 5 or fewer entries per project: present as-is
- More than 5: consolidate into themes, keep ticket/PR references that are discrete deliverables
- Drop process noise ("ran daily report")
- Target 10-20 lines total

### 5. Format

```
### Standup

**Last session** (YYYY-MM-DD, DayOfWeek):
_[project-a]_
- Summary item 1
- Summary item 2

_[project-b]_
- Summary item 3
```

Use `date -d 'YYYY-MM-DD' +%A` for the day-of-week label. If all entries belong to one project, skip the project subheading.

## Notes

- Apply the ghostwriter skill for tone
- Read-only, never modifies vault content
- Brief and scannable, this is a communication artifact
