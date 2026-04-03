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

All notes files:
!`find notes/ -name "*.md" 2>/dev/null | sort`

**If FIRST_COMPILE:** process ALL notes files listed above.

**Otherwise:** use the Bash tool to run `git diff <last-hash>..HEAD --name-only -- notes/` (substituting the hash from .wikime) to find which notes changed. If no notes changed, respond "No notes changes to compile." and stop.

## Step 2: Read changed notes

Read each changed note file in full. You need full context to extract topics correctly — not just the diff lines.

## Step 3: Read existing wiki state

Use Glob to find `wiki/**/_index.md` files. Read `wiki/_index.md` if it exists — this is your map of existing topics and articles.

## Step 4: Plan updates

For each piece of content in the notes, decide:
- **Existing article?** → Read it, merge new information, update `last_updated` and `sources` frontmatter.
- **New article needed?** → Plan the topic folder and article filename.
- **New cross-links?** → Plan markdown links between related articles.

Group related content. One note entry might touch multiple articles. Multiple note entries might feed into one article.

## Step 5: Write wiki updates

For each article to create or update:

**New articles** must follow this format:
```yaml
---
title: Article Title
topic: topic-folder-name
sources:
  - notes/YYYY/MM-wN.md
last_updated: YYYY-MM-DD
---
```

Include:
- A clear synthesis of the information (not just copied text from notes)
- `## Key Takeaways` section with bullet points
- `## Related` section with standard markdown links using relative paths

**Topic _index.md files** list all articles in the topic:
```markdown
# Topic Name

Brief description of this topic area.

## Articles
- [Article Title](article-name.md) — one-line summary
```

**Master wiki/_index.md** lists all topics:
```markdown
# Wiki Index

## Topics
- [Topic Name](topic-name/_index.md) — brief topic description (N articles)
```

Create topic directories with `mkdir -p` before writing files.

## Step 6: Update .wikime and commit

1. Write the current HEAD commit hash to `.wikime`
2. Stage all changes: `git add wiki/ .wikime`
3. Commit with message: `wikime: compile — updated N articles, created M new`

## Rules

- **Synthesize, don't copy.** Wiki articles should be well-written summaries, not copy-paste from notes.
- **Merge, don't duplicate.** If a topic already has an article, update it rather than creating a new one.
- **Kebab-case** for all folder and file names.
- **Links** use standard markdown relative links: `[Title](relative/path.md)`.
- **Every article** needs frontmatter with title, topic, sources, last_updated.
- **Every topic folder** needs an `_index.md`.
- **Master index** (`wiki/_index.md`) must be kept up to date.
