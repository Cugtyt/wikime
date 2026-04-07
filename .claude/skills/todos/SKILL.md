---
name: todos
description: Scan notes for open action items and TODO-like patterns. Shows what's outstanding and what appears to be done. Read-only — no files modified.
allowed-tools: Read, Glob, Grep
---

# TODOs

Scan all notes for action items and show their status.

## Context

Available notes:
!`find notes/ -name "*.md" 2>/dev/null | sort`

Today:
!`date +%Y-%m-%d`

## Steps

1. **Read all note files.**

2. **Extract action items.** Look for patterns like:
   - "need to", "needs to", "TODO", "todo"
   - "follow up with", "follow up on"
   - "should", "have to", "must"
   - "waiting on", "waiting for"
   - "blocked by", "blocked on"
   - Action verbs at the start of a bullet: "write", "fix", "update", "send", "schedule", "review", "set up"

3. **Infer completion.** For each action item, scan later notes for evidence it was done:
   - "finished", "completed", "done", "fixed", "sent", "wrote", "resolved"
   - The same topic mentioned in a past-tense or completion context
   - Use your judgment — if a later note says "migrated to Postgres" then "need to write migration plan" is likely done

4. **Output to terminal** in this format:

```
## Open
- [ ] item description — (source: notes/2026/04-w1.md, Mon Apr 1)
- [ ] item description — (source: notes/2026/04-w1.md, Wed Apr 3)

## Likely Done
- [x] item description — (source: notes/2026/04-w1.md, Tue Apr 2) — appears resolved in notes/2026/04-w2.md

## Summary
N open items, M likely completed
```

Sort open items by date, newest first. Do not create or modify any files.
