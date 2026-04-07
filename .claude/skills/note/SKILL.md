---
name: note
description: Quick-add a note to the current week's file. Auto-detects week, creates file if needed, appends under today's date section, formats into clean bullets. Usage - /note <what happened>
argument-hint: <what happened today>
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git add*), Bash(git commit*), Bash(mkdir*), Bash(date*), Bash(rm .wikime-cache*)
---

# Add a Note

You are the wikime note-taker. Append the user's input to the correct weekly note file.

**Input:** $ARGUMENTS

## Step 1: Determine the current week file

Today:
!`date +"%A %Y-%m-%d"`

Use the Bash tool to find the Monday of the current week and derive the file path:
1. Find the most recent Monday (including today if it's Monday)
2. Get its month (zero-padded) and ISO week number (`date +%V` on that Monday)

File path: `notes/YYYY/MM-wWW.md` (MM = month of Monday, WW = ISO week number zero-padded)

Examples:
- Monday is 2026-01-05 (week 2) → `notes/2026/01-w02.md`
- Monday is 2026-03-30 (week 14) → `notes/2026/03-w14.md`
- Monday is 2026-04-06 (week 15) → `notes/2026/04-w15.md`
- Monday is 2026-12-28 (week 53) → `notes/2026/12-w53.md`

## Step 2: Read or create the file

Check if the file exists using Read.

**If it doesn't exist:**
- Create the directory with `mkdir -p notes/YYYY/`
- Create the file with a header:
```markdown
# Week N - Month YYYY

```

**If it exists:** read it to find where to append.

## Step 3: Find or create today's section

Look for a section matching today: `## DayName YYYY-MM-DD` (e.g. `## Thursday 2026-04-03`).

**If the section exists:** append the new bullets after the last line of that section (before the next `##` or end of file).

**If the section doesn't exist:** add it at the end of the file:
```markdown

## DayName YYYY-MM-DD

```

## Step 4: Check cache and combine input

Read `.wikime-cache` if it exists. This file contains quick-capture entries from the `wikime add` CLI, one per line in format `[YYYY-MM-DD HH:MM] text`.

Combine cache entries with the user's `$ARGUMENTS` input. Group cache entries by their timestamp date — entries from today go under today's section, entries from previous days go under their respective day sections.

After processing, delete `.wikime-cache` using Bash: `rm .wikime-cache`

## Step 5: Format and append

Take all input (cache + user's text) and format into clean, consistent bullet points:
- Split into separate bullet points by topic/event
- Start each bullet with `- `
- Format code, tool names, and commands with backticks
- Expand abbreviations if obvious (e.g. "k8s" → "Kubernetes")
- Keep each bullet concise — one idea per line
- Preserve the user's meaning exactly — don't add information

Append the formatted bullets under today's section.

## Step 6: Impact check (analysis only — don't present yet)

For each NEW bullet just added (not previous days' items), classify it per `CLAUDE.md` Impact Context rules. Do this silently — you'll present results in Step 6.

**Routine** (attended standup, reviewed PR, read docs) → skip, no impact needed.

**Significant** → check if the wiki has a related topic:

1. Read `wiki/_index.md`. Use Grep to quickly scan wiki article titles for related terms.
2. Note which situation applies for each significant bullet:

| Situation | What to present in Step 7 |
|-----------|--------------------------|
| Wiki has a related topic with impact context | Propose: "This connects to [Topic] — [existing impact summary]. Sound right?" |
| Wiki has a related topic, no impact context | Ask: "This connects to [Topic] but we don't have the impact story. What's the goal? Who benefits?" |
| No related wiki topic | Ask: "This looks significant. What's the impact? (skip if not important)" |

**Keep it fast:** read `wiki/_index.md` and at most 1-2 relevant articles. Don't scan the whole wiki.

**Never re-ask** about items from previous days. Only check the new bullets from this invocation.

## Step 7: Present everything, confirm, and commit

Show the user what was appended AND any impact questions from Step 6, all in one message:
```
Added to notes/2026/03-w5.md under ## Thursday 2026-04-03:
- Fixed Flannel MTU across staging
- Attended team standup

💡 Impact check:
- "Fixed Flannel MTU across staging" connects to Cluster Stability — unblocking 3 teams. Sound right?
(reply with context to add, adjust, or "skip")
```

If the user provides impact context, append it to the relevant bullet with a ` — ` separator (e.g. `- Fixed Flannel MTU — unblocked 3 teams dependent on staging cluster`). If they say "skip", proceed as-is.

Then ask to commit:
```
Commit this change?
```

Wait for user response. If they approve, commit:
```bash
git add notes/
git commit -m "note: YYYY-MM-DD — brief summary of what was added"
```

If they want changes, edit and re-present.
