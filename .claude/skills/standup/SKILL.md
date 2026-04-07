---
name: standup
description: Generate a standup summary from recent notes. Shows what you did yesterday, plan for today, and any blockers. Read-only — no files modified.
allowed-tools: Read, Glob, Grep
---

# Standup

Generate a standup summary from recent notes.

## Context

Today:
!`date +"%A %Y-%m-%d"`

Available notes:
!`find notes/ -name "*.md" 2>/dev/null | sort`

## Steps

1. **Find yesterday's and today's notes.** Read the most recent 2 note files from the list above (the current week's file and possibly the previous week's). Scan for:
   - **Yesterday:** If today is Monday, look for Friday's section (`## Friday YYYY-MM-DD`). Otherwise look for the previous weekday's section. This may be in the previous week's file.
   - **Today:** Look for today's section (`## DayName YYYY-MM-DD`) in the current week's file.

2. **Generate the standup** in this format:

```
## Yesterday
- What you accomplished (extracted from yesterday's note section)

## Today
- What's planned (extracted from today's note section, if it exists)
- If no today section exists, say "No plan logged yet — use /note to add one."

## Blockers
- Any blockers, open questions, or waiting-on-others items
- Infer from phrases like "blocked by", "waiting on", "need to", "can't proceed"
- If none found, say "None"
```

3. **Output to terminal only.** Do not create or modify any files.

Keep it concise — bullet points, no prose.
