---
name: report
description: Generate a weekly report from notes. Defaults to current week, accepts argument like "/report last week". Read-only — no files modified. Usage - /report [this week|last week|YYYY/MM-wN]
argument-hint: [this week | last week | YYYY/MM-wN]
allowed-tools: Read, Glob, Grep
---

# Weekly Report

Generate a structured weekly report from notes.

## Context

Today:
!`date +"%A %Y-%m-%d"`

Available notes:
!`find notes/ -name "*.md" 2>/dev/null | sort`

**Time range:** $ARGUMENTS

If no argument given, default to "this week". If "last week", use the previous week's note file. If a specific file reference like `2026/04-w1`, use that.

## Steps

1. **Determine which note file(s) to read** based on the time range.

2. **Read the note file(s).**

3. **Also read `wiki/_index.md`** if it exists, to add topic context to the report.

4. **Generate the report** in this format:

```
# Weekly Report — Week of [date range]

## Accomplishments
- Key things completed this week, grouped by topic
- Focus on outcomes, not activity

## Key Decisions
- Decisions made, who was involved, rationale
- Only include if decisions were actually made

## Blockers & Open Items
- What's still unresolved
- What you're waiting on

## Next Week
- What's planned or expected
- Infer from open items, upcoming deadlines, ongoing work

## People
- Who you interacted with and on what topics
- Useful for 1:1 prep
```

5. **Output to terminal only.** Do not create or modify any files.

If the user wants to save it, suggest: "Want me to save this? I can write it to `notes/YYYY/MM-wN-report.md`."
