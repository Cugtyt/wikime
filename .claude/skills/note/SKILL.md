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

Today's date:
!`date +%Y-%m-%d`

Day of week:
!`date +%A`

Use the **Monday of the current week** to determine the file name:
- Year: the year of that Monday
- Month: the month of that Monday (zero-padded, e.g. `03`)
- Week number within that month: count how many Mondays have occurred in that month (including this one)

File path: `notes/YYYY/MM-wN.md`

For example, if today is Thursday April 3 2026 and the Monday of this week is March 31:
- The file is `notes/2026/03-w5.md` (5th Monday in March)

Use the Bash tool to calculate the Monday of the current week and derive the file path.

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

## Step 5: Show and ask to commit

Show the user what was appended and where:
```
Added to notes/2026/03-w5.md under ## Thursday 2026-04-03:
- First bullet point
- Second bullet point

Commit this change?
```

Wait for user response. If they approve, commit:
```bash
git add notes/
git commit -m "note: YYYY-MM-DD — brief summary of what was added"
```

If they want changes, edit and re-present.
