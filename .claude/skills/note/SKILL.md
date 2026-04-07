---
name: note
description: Quick-add a note to the current week's file. Auto-detects week, creates file if needed, appends under today's date section, formats into clean bullets. Usage - /note <what happened>
argument-hint: <what happened today>
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git add*), Bash(git commit*), Bash(mkdir*), Bash(date*)
---

# Add a Note

You are the wikime note-taker. Append the user's input to the correct weekly note file.

**Input:** $ARGUMENTS

## Step 1: Determine the current week file

Today:
!`date +"%A %Y-%m-%d"`

Use the Bash tool to find the Monday of the current week and derive the file path:
1. Find the most recent Monday (including today if it's Monday)
2. Get its year, month (zero-padded), and week-of-month (day 1-7 = w1, 8-14 = w2, 15-21 = w3, 22-28 = w4, 29-31 = w5)

File path: `notes/YYYY/MM-wW.md`

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

## Step 4: Format and append

Take the user's raw input and format it into clean, consistent bullet points:
- Split into separate bullet points by topic/event
- Start each bullet with `- `
- Format code, tool names, and commands with backticks
- Expand abbreviations if obvious (e.g. "k8s" → "Kubernetes")
- Keep each bullet concise — one idea per line
- Preserve the user's meaning exactly — don't add information

Append the formatted bullets under today's section.

## Step 5: Impact check

For each NEW bullet just added (not previous days' items), classify it per `CLAUDE.md` Impact Context rules:

**Routine** (attended standup, reviewed PR, read docs) → skip, no impact needed.

**Significant** → check if the wiki has a related topic:

1. Read `wiki/_index.md`. Use Grep to quickly scan wiki article titles for related terms.
2. Based on what you find:

| Situation | Action |
|-----------|--------|
| Wiki has a related topic with impact context | Propose: "This connects to [Topic] — [existing impact summary]. Sound right?" |
| Wiki has a related topic, no impact context | Ask: "This connects to [Topic] but we don't have the impact story. What's the goal? Who benefits?" |
| No related wiki topic | Ask: "This looks significant. What's the impact? (skip if not important)" |

**Keep it fast:** read `wiki/_index.md` and at most 1-2 relevant articles. Don't scan the whole wiki.

**Never re-ask** about items from previous days. Only check the new bullets from this invocation.

## Step 6: Show, confirm, and commit

Show the user what was appended:
```
Added to notes/2026/03-w5.md under ## Thursday 2026-04-03:
- First bullet point
- Second bullet point
```

If impact questions were raised in Step 5, present them now. If the user provides impact context, append it to the relevant bullet with a ` — ` separator (e.g. `- Fixed Flannel MTU — unblocked 3 teams dependent on staging cluster`). If they say "skip", proceed as-is.

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
