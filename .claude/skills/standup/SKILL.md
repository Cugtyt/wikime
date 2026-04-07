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

1. **Find yesterday's notes.** If today is Monday, "yesterday" means Friday. Find the note file that contains the most recent working day's section.

2. **Find today's notes.** Check if today's date section exists in the current week's note file.

3. **Read the relevant note files.**

4. **Generate the standup** in this format:

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

5. **Output to terminal only.** Do not create or modify any files.

Keep it concise — bullet points, no prose.
