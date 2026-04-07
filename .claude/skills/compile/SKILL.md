---
name: compile
description: Compile notes into wiki. Reads git diff since last compile, extracts topics from changed notes, creates/updates wiki articles, indexes, and backlinks. Auto-commits results.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git diff*), Bash(git log*), Bash(git rev-parse*), Bash(git add*), Bash(git commit*), Bash(mkdir*)
---

# Compile Notes into Wiki

You are the wikime compiler. Your job is to read changed notes and update the wiki.

## Step 1: Determine what changed

Read `.wikime` to get the last compiled commit hash. If the file doesn't exist, this is a first compile — all notes should be processed.

Last compiled commit (empty means first compile):
!`cat .wikime 2>/dev/null || echo FIRST_COMPILE`

Current HEAD:
!`git rev-parse HEAD`

Weekly note files (the only input — ignore any other files in notes/):
!`find notes/ -name "*.md" 2>/dev/null | sort`

**If FIRST_COMPILE:** process ALL weekly note files listed above.

**Otherwise:** use the Bash tool to run `git diff <last-hash>..HEAD --name-only -- notes/` (substituting the hash from .wikime) to find which notes changed. If no notes changed, respond "No notes changes to compile." and stop.

## Step 2: Read changed notes

Read each changed note file in full. You need full context to extract topics correctly — not just the diff lines.

## Step 3: Read existing wiki state

Use Glob to find `wiki/**/_index.md` files. Read `wiki/_index.md` if it exists — this is your map of existing topics and articles.

## Step 4: Filter — what is wiki material?

Read the "What belongs in the wiki" section in `CLAUDE.md`. For each item in the changed notes, decide:

- **Wiki material** (incidents, decisions, migrations, runbooks, real learnings) → proceed to planning
- **Not wiki material** (routine tasks, meetings, one-off conversations, ephemeral updates) → skip, leave in notes only

Be selective. Not every note becomes a wiki article. The wiki should feel like an engineering reference, not a log.

## Step 5: Plan updates

For the items that passed the filter:
- **Existing article?** → Read it, merge new information, update `last_updated` and `sources` frontmatter.
- **New article needed?** → Plan the topic folder and article filename.
- **New cross-links?** → Plan links only if the relationship is concrete enough to name (see CLAUDE.md "Related — named relationships only").

Group related content. One note entry might touch multiple articles. Multiple note entries might feed into one article.

## Step 6: Confirm or proceed

Review your plan from Step 4. If every decision is clear and unambiguous — obvious topic assignments, straightforward merges into existing articles — proceed directly to Step 6 without asking.

**But if any decision requires judgment, stop and ask the user.** Examples:
- A note could belong to multiple topics and it's not obvious which
- Two existing articles could both be the merge target
- A note covers something entirely new and the topic name/structure isn't obvious
- Content seems contradictory to what's already in the wiki

When asking, present your plan as a summary:
```
Compile plan:
- CREATE: wiki/topic/article-name.md — brief description
- UPDATE: wiki/topic/existing-article.md — what's being added
- QUESTION: "This note about X could go under topic-a or topic-b — which do you prefer?"
```

Wait for user input before proceeding. If the user gives feedback, adjust and re-present if needed.

## Step 7: Write wiki updates

Follow the wiki article format and conventions defined in `CLAUDE.md`. Read it first if you haven't already.

Create topic directories with `mkdir -p` before writing files.

**Key rules:**
- **Outcome-first.** First paragraph states what happened, why it mattered, who it affected. See CLAUDE.md for examples.
- **Merge, don't duplicate.** If a topic already has an article, update it rather than creating a new one.
- **Named relationships only.** Only add links to `## Related` if you can name the relationship type (dependency, used-by, decision, same-initiative, operational-support). See CLAUDE.md.
- **Preserve impact context.** When notes include who was affected, what improved, or why something matters, carry that into the wiki article prominently.
- Every topic folder needs an `_index.md`. Master `wiki/_index.md` must be kept up to date.

## Step 8: Archive check

After writing updates, check if any topics should be archived. Follow the Wiki Archiving rules in `CLAUDE.md`. If all articles in a topic have `last_updated` older than 3 months and no new notes reference them, suggest moving the topic to `wiki/archive/`. Ask the user before archiving — don't do it automatically.

## Step 9: Update .wikime and commit

1. Write the current HEAD commit hash to `.wikime`
2. Stage all changes: `git add wiki/ .wikime`
3. Commit with message: `wikime: compile — updated N articles, created M new`

Compile complete. Run `/lint` to check wiki health.
