---
name: compile
description: Compile notes into wiki. Reads git diff since last compile, extracts topics from changed notes, creates/updates wiki articles, indexes, and backlinks. Auto-commits results.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git diff*), Bash(git log*), Bash(git rev-parse*), Bash(git add*), Bash(git commit*), Bash(mkdir*)
---

# Compile Notes into Wiki

You are the wikime compiler. Your job is to read changed notes and update the wiki.

## Step 1: Determine what changed

Read `.wikime` to get the last compiled commit hash. If the file doesn't exist, this is a first compile — all notes should be processed.

Changed files since last compile:
!`cat .wikime 2>/dev/null && echo "---LAST_HASH---" || echo "FIRST_COMPILE"`
!`git rev-parse HEAD`
!`git diff $(cat .wikime 2>/dev/null || echo "4b825dc642cb6eb9a060e54bf899d15363da7b23")..HEAD --name-only -- notes/`

If the output after FIRST_COMPILE or ---LAST_HASH--- shows no changed files in notes/, respond "No notes changes to compile." and stop.

## Step 2: Read changed notes

Read each changed note file in full. You need full context to extract topics correctly — not just the diff lines.

## Step 3: Read existing wiki state

Use Glob to find `wiki/**/_index.md` files. Read `wiki/_index.md` if it exists — this is your map of existing topics and articles.

## Step 4: Plan updates

For each piece of content in the notes, decide:
- **Existing article?** → Read it, merge new information, update `last_updated` and `sources` frontmatter.
- **New article needed?** → Plan the topic folder and article filename.
- **New cross-links?** → Plan wikilinks between related articles.

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
- `## Related` section with `[[topic/article-name]]` wikilinks

**Topic _index.md files** list all articles in the topic:
```markdown
# Topic Name

Brief description of this topic area.

## Articles
- [[topic/article-name]] — one-line summary
```

**Master wiki/_index.md** lists all topics:
```markdown
# Wiki Index

## Topics
- [[topic-name/_index]] — brief topic description (N articles)
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
- **Wikilinks** use `[[topic/article-name]]` format (no .md extension).
- **Every article** needs frontmatter with title, topic, sources, last_updated.
- **Every topic folder** needs an `_index.md`.
- **Master index** (`wiki/_index.md`) must be kept up to date.
