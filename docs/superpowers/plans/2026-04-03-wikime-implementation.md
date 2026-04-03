# wikime Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build three Claude Code slash commands (`/compile`, `/query`, `/lint`) that maintain a topical wiki from weekly work notes, all git-native.

**Architecture:** Claude Code skills in `.claude/skills/` with markdown prompt files. Each skill uses `!` backtick syntax for dynamic shell injection (git diff, file listing) and Claude Code's built-in tools (Read, Write, Edit, Glob, Grep, Bash) for file operations. State tracked via `.wikime` file containing last compiled commit hash.

**Tech Stack:** Claude Code skills (markdown), Bash (git operations), Markdown (all data)

---

## File Structure

```
.claude/
  skills/
    compile/
      SKILL.md              ← /compile slash command
    query/
      SKILL.md              ← /query slash command
    lint/
      SKILL.md              ← /lint slash command
notes/
  2026/
    04-w1.md                ← sample weekly note (for testing)
wiki/
  _index.md                 ← master index (created by first compile)
.wikime                     ← last compiled commit hash
.gitignore                  ← already exists
CLAUDE.md                   ← project instructions for Claude Code
```

**Responsibilities:**
- `compile/SKILL.md` — reads git diff, reads/writes wiki articles, updates indexes, commits
- `query/SKILL.md` — routes questions to wiki or notes, reads relevant files, answers in markdown
- `lint/SKILL.md` — scans wiki for broken links, orphans, staleness, duplicates; reports and auto-fixes

---

### Task 1: Project Scaffolding & CLAUDE.md

**Files:**
- Create: `CLAUDE.md`
- Create: `notes/2026/04-w1.md` (sample note for testing)
- Create: `.claude/skills/` directory structure

- [ ] **Step 1: Create CLAUDE.md with project conventions**

```markdown
# wikime

LLM-maintained personal knowledge base. Weekly work notes in `notes/` are compiled into a topical wiki in `wiki/` via slash commands.

## Structure

- `notes/YYYY/MM-wN.md` — weekly notes (human-written)
- `wiki/` — topical articles (LLM-maintained, never edit manually)
- `wiki/_index.md` — master index of all topics
- `wiki/<topic>/_index.md` — per-topic index
- `.wikime` — last compiled commit hash

## Wiki Article Format

All wiki articles must use this format:

```yaml
---
title: Article Title
topic: topic-folder-name
sources:
  - notes/2026/04-w1.md
last_updated: YYYY-MM-DD
---
```

- Use `[[topic/article-name]]` wikilinks for cross-references
- Each article has a `## Related` section with wikilinks
- Each article has a `## Key Takeaways` section

## Commands

- `/compile` — compile notes diff into wiki
- `/query <question>` — ask questions against the knowledge base
- `/lint` — health check the wiki

## Conventions

- All content is markdown
- Obsidian-compatible wikilinks: `[[topic/article-name]]`
- Topic folders are lowercase kebab-case
- Article filenames are lowercase kebab-case
- _index.md files list all articles in their directory with one-line summaries
```

Write this to `CLAUDE.md` in the project root.

- [ ] **Step 2: Create a sample weekly note for testing**

```markdown
# Week 1 - April 2026

## Monday 2026-04-01

- Started investigating Kubernetes CNI issues in the staging cluster
- The Flannel pods keep crashing with OOMKilled errors
- Found that the MTU settings were mismatched between nodes
- Talked to Sarah about migrating the auth service to PostgreSQL — she's in favor

## Tuesday 2026-04-02

- Fixed the Flannel MTU issue by setting `--iface=eth0` explicitly
- Wrote a runbook for CNI debugging: check MTU, check iptables rules, check pod CIDR allocation
- Started reviewing the PostgreSQL migration plan — main concern is the session table schema
- Python async patterns: learned about `asyncio.TaskGroup` for structured concurrency

## Wednesday 2026-04-03

- Team decided to go ahead with PostgreSQL migration, targeting end of April
- Need to handle the session token format change for compliance (legal flagged the old format)
- Helped onboard Jake to the Kubernetes monitoring stack — showed him Grafana dashboards
- More async Python: `asyncio.Semaphore` for rate limiting API calls
```

Write this to `notes/2026/04-w1.md`.

- [ ] **Step 3: Create skill directories**

Run:
```bash
mkdir -p .claude/skills/compile .claude/skills/query .claude/skills/lint
```

- [ ] **Step 4: Commit scaffolding**

```bash
git add CLAUDE.md notes/2026/04-w1.md .claude/skills/
git commit -m "Add project scaffolding: CLAUDE.md, sample note, skill dirs"
```

---

### Task 2: `/compile` Skill

**Files:**
- Create: `.claude/skills/compile/SKILL.md`

This is the core skill. It reads the git diff since last compile, processes changed notes, and creates/updates wiki articles.

- [ ] **Step 1: Write the /compile skill**

```markdown
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
```

Write this to `.claude/skills/compile/SKILL.md`.

- [ ] **Step 2: Test /compile with the sample note**

Run `/compile` in Claude Code. Verify:
1. It reads `notes/2026/04-w1.md`
2. It creates wiki articles (expect topics like: kubernetes, python, decisions)
3. It creates `wiki/_index.md` with all topics listed
4. It creates topic `_index.md` files
5. Articles have correct frontmatter with `sources: [notes/2026/04-w1.md]`
6. Articles have wikilinks in `## Related` sections
7. `.wikime` contains the current HEAD hash
8. Changes are auto-committed

- [ ] **Step 3: Verify git state**

Run:
```bash
git log --oneline -3
cat .wikime
ls wiki/
```

Expected: the compile commit appears in git log, `.wikime` has the commit hash, `wiki/` has topic folders.

- [ ] **Step 4: Commit any fixes**

If testing revealed issues with the skill, fix `.claude/skills/compile/SKILL.md` and commit:
```bash
git add .claude/skills/compile/SKILL.md
git commit -m "Fix /compile skill after testing"
```

---

### Task 3: `/query` Skill

**Files:**
- Create: `.claude/skills/query/SKILL.md`

- [ ] **Step 1: Write the /query skill**

```markdown
---
name: query
description: Ask questions against the wikime knowledge base. Routes to wiki for topical questions, notes for temporal questions. Usage - /query <your question>
argument-hint: <your question>
allowed-tools: Read, Glob, Grep, Bash(git log*)
---

# Query the Knowledge Base

You are the wikime query engine. Answer the user's question using the knowledge base.

**Question:** $ARGUMENTS

## Routing

Determine the query type from the question:

**Topical** (mentions concepts, tools, technologies, decisions):
→ Read `wiki/_index.md` to find relevant topics, then read the relevant articles.

**Temporal** (mentions dates, "this week", "last Monday", "recently", "today"):
→ Read notes directly. Use Glob to find `notes/**/*.md` and read the relevant time period.
→ For "this week", determine the current week number and read that file.
→ For "last week", read the previous week's file.

**Mixed** (e.g., "when did I last work on X?"):
→ First find the topic in wiki, check its `sources` frontmatter for which notes contributed, then read those notes for dates and details.

## Answering

- Answer in markdown format directly in the conversation.
- Cite your sources: mention which notes or wiki articles the answer came from.
- If you can't find relevant information, say so clearly.
- Keep answers concise and focused on what was asked.
- If the question is ambiguous, answer your best interpretation and note the assumption.

## Context

Current date for temporal reference:
!`date +%Y-%m-%d`

Available wiki topics:
!`ls wiki/ 2>/dev/null || echo "Wiki is empty — run /compile first"`

Available notes:
!`find notes/ -name "*.md" 2>/dev/null | sort || echo "No notes found"`
```

Write this to `.claude/skills/query/SKILL.md`.

- [ ] **Step 2: Test /query with a topical question**

Run: `/query what do I know about Kubernetes CNI issues?`

Verify:
1. It reads `wiki/_index.md` and finds the kubernetes topic
2. It reads the relevant article(s)
3. It returns a markdown answer citing the wiki article
4. Answer includes the Flannel MTU fix and the debugging runbook

- [ ] **Step 3: Test /query with a temporal question**

Run: `/query what did I do on Tuesday April 2nd?`

Verify:
1. It reads `notes/2026/04-w1.md` directly
2. It returns details from Tuesday's entries
3. Answer cites `notes/2026/04-w1.md`

- [ ] **Step 4: Test /query with a mixed question**

Run: `/query when did I last work on PostgreSQL migration?`

Verify:
1. It finds the relevant wiki article first
2. It follows `sources` to the notes
3. It returns dates and details from the notes

- [ ] **Step 5: Commit**

```bash
git add .claude/skills/query/SKILL.md
git commit -m "Add /query skill for knowledge base Q&A"
```

---

### Task 4: `/lint` Skill

**Files:**
- Create: `.claude/skills/lint/SKILL.md`

- [ ] **Step 1: Write the /lint skill**

```markdown
---
name: lint
description: Health check the wikime knowledge base. Finds broken links, orphan articles, stale content, missing coverage, and duplicate concepts. Auto-fixes simple issues.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git add*), Bash(git commit*)
---

# Lint the Knowledge Base

You are the wikime linter. Scan the wiki for quality issues and fix what you can.

## Available data

Wiki articles:
!`find wiki/ -name "*.md" 2>/dev/null | sort || echo "No wiki found — run /compile first"`

Notes files:
!`find notes/ -name "*.md" 2>/dev/null | sort || echo "No notes found"`

## Checks to perform

Read all wiki markdown files, then check for:

### 1. Broken Wikilinks
Scan all articles for `[[...]]` wikilinks. For each one, verify the target file exists at `wiki/<link>.md`. Report any broken links.

**Auto-fix:** Remove broken wikilinks from `## Related` sections.

### 2. Orphan Articles
Find wiki articles (excluding `_index.md` files) that are not linked from any other article's wikilinks or `_index.md`. Every article should be reachable from the master index.

**Auto-fix:** Add missing entries to the relevant `_index.md`.

### 3. Stale Articles
Compare each article's `last_updated` frontmatter date with the modification dates of its `sources` notes. If a source note has been updated more recently than the article's `last_updated`, it's stale.

**Flag for review:** List stale articles and suggest running `/compile` to update them.

### 4. Missing Coverage
Scan all notes for recurring topics, tools, or concepts that appear 3+ times but have no wiki article. Use Grep to find frequently mentioned terms across notes.

**Flag for review:** List potential new articles with the note references.

### 5. Duplicate Concepts
Look for wiki articles with very similar titles or overlapping content. Check if two articles cover the same concept under different names.

**Flag for review:** List potential duplicates with a merge suggestion.

### 6. Index Consistency
Verify that `wiki/_index.md` lists all topic folders, and each topic `_index.md` lists all articles in its folder.

**Auto-fix:** Update index files to include missing entries.

## Output

Print a markdown report with sections for each check:

```
# Wiki Health Report

## Broken Wikilinks
- ✅ No broken links (or list them)

## Orphan Articles
- ✅ All articles linked (or list orphans)

## Stale Articles
- ⚠️ wiki/kubernetes/cni-debugging.md — last_updated 2026-04-01, source notes/2026/04-w2.md modified since

## Missing Coverage
- 💡 "grafana" mentioned 4 times in notes but has no wiki article

## Duplicate Concepts
- ✅ No duplicates found (or list them)

## Index Consistency
- ✅ All indexes up to date (or list fixes applied)

## Summary
X issues found, Y auto-fixed, Z need review.
```

If any auto-fixes were applied, stage and commit:
```bash
git add wiki/
git commit -m "wikime: lint — auto-fixed N issues"
```
```

Write this to `.claude/skills/lint/SKILL.md`.

- [ ] **Step 2: Test /lint against current wiki**

Run `/lint` in Claude Code. Verify:
1. It scans all wiki files
2. It produces a markdown health report
3. It catches any issues introduced during compile testing
4. Auto-fixes are applied and committed (if any)

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/lint/SKILL.md
git commit -m "Add /lint skill for wiki health checks"
```

---

### Task 5: End-to-End Verification

**Files:** None created — this is a verification task.

- [ ] **Step 1: Add new content to the weekly note**

Append to `notes/2026/04-w1.md`:

```markdown

## Thursday 2026-04-04

- Deep dive into Python type hints — learned about `Protocol` classes for structural subtyping
- The PostgreSQL migration needs a rollback plan — discussed with Sarah
- Kubernetes: set up horizontal pod autoscaler for the auth service
- Started documenting our API rate limiting approach using asyncio.Semaphore
```

- [ ] **Step 2: Commit the note update**

```bash
git add notes/2026/04-w1.md
git commit -m "Add Thursday notes"
```

- [ ] **Step 3: Run /compile and verify incremental update**

Run `/compile`. Verify:
1. It detects only `notes/2026/04-w1.md` changed
2. It updates existing articles (kubernetes, python, decisions) with new info
3. It does NOT recreate articles from scratch — it merges
4. `last_updated` and `sources` frontmatter are correct
5. New wikilinks added if appropriate
6. Auto-committed

- [ ] **Step 4: Run /query to verify the new content is accessible**

Run: `/query what's the rollback plan for the PostgreSQL migration?`

Verify it finds the info from Thursday's notes, synthesized through the wiki.

- [ ] **Step 5: Run /lint for final health check**

Run `/lint`. Verify clean report or only minor issues.

- [ ] **Step 6: Final commit if needed**

If any fixes were needed during verification:
```bash
git add -A
git commit -m "wikime: end-to-end verification fixes"
```
