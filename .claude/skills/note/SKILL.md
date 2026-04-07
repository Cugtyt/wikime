---
name: note
description: Quick-add a note to the current week's file. Auto-detects week, creates file if needed, appends under today's date section, formats into clean bullets. Usage - /note <what happened>
argument-hint: <what happened today>
allowed-tools: Read, Write, Edit, Glob, Bash(git add*), Bash(git commit*), Bash(mkdir*), Bash(date*)
---

# Add a Note

You are the wikime note-taker. Append the user's input to the correct weekly note file.

**Input:** $ARGUMENTS

## Step 1: Determine the current week file

Today:
!`date +"%A %Y-%m-%d"`

Monday of this week (year, month, day, week-of-month):
!`python3 -c "from datetime import date,timedelta;t=date.today();m=t-timedelta(days=t.weekday());w=(m.day-1)//7+1;print(f'{m.year} {m.month:02d} {m.day:02d} {w}')" 2>/dev/null || python -c "from datetime import date,timedelta;t=date.today();m=t-timedelta(days=t.weekday());w=(m.day-1)//7+1;print(f'{m.year} {m.month:02d} {m.day:02d} {w}')"` 

Read the output above as: `YYYY MM DD W`

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

Before presenting to the user, review **the entire week's note file** (not just the new bullets) for items that look like significant work but lack impact context.

**Significant signals:** fixing outages, launching features, unblocking teams, making decisions, completing migrations, setting up infrastructure, resolving incidents, onboarding people.

**Has impact context:** mentions who was affected, what improved, why it matters, numbers, or outcomes.

**Lacks impact context:** just states what was done without the "so what."

Ignore routine items (attended standup, reviewed PR, read docs) — these don't need impact.

## Step 6: Show, nudge, and ask to commit

Show the user what was appended:
```
Added to notes/2026/03-w5.md under ## Thursday 2026-04-03:
- First bullet point
- Second bullet point
```

If any items this week (new or from previous days) are missing impact context, nudge:
```
💡 A few items this week could use impact context (helps with /review later):
- [Mon] Fixed the Flannel MTU issue across staging — who was affected? what was the impact?
- [Thu] Set up HPA for the auth service — why? what does this improve?
(reply with context to add, or "skip")
```

If the user provides context, update those bullets in the note file to include it. If they say "skip", proceed.

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
