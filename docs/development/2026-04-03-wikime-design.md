# wikime — LLM-Maintained Personal Knowledge Base

**Date:** 2026-04-03
**Status:** Approved
**Inspired by:** Andrej Karpathy's X post on LLM knowledge bases

## Summary

wikime is a git-native personal knowledge base where you write weekly work notes in markdown, and an LLM (Claude Code) compiles them into a topical wiki. The wiki is maintained entirely by the LLM — you rarely touch it directly. All operations are Claude Code slash commands: `/compile`, `/query`, `/lint`.

## Repository Structure

```
wikime/
  notes/                    ← you write here
    2026/
      04-w1.md              ← weekly note (April week 1)
      04-w2.md
      ...
  wiki/                     ← LLM maintains this
    _index.md               ← master index of all topics
    kubernetes/
      _index.md             ← topic index
      cni-debugging.md
      helm-patterns.md
    python/
      _index.md
      async-patterns.md
    decisions/
      _index.md
      migrate-to-postgres.md
  .wikime                   ← last compiled commit hash
  .obsidian/                ← Obsidian config (optional)
```

### Two layers

- **`notes/`** — temporal layer. Weekly markdown files organized as `notes/YYYY/MM-wN.md`. You write and commit freely.
- **`wiki/`** — topical layer. Articles organized by topic, maintained by the LLM. Each topic has a folder with an `_index.md` and article files.

Backlinks bridge the two layers: wiki articles reference their source notes, and temporal queries can follow links back to topical articles.

## Wiki Article Format

```markdown
---
title: CNI Debugging
topic: kubernetes
sources:
  - notes/2026/04-w1.md
  - notes/2026/03-w4.md
last_updated: 2026-04-03
---

# CNI Debugging

Synthesized content from multiple weekly notes.

## Key Takeaways
- ...

## Related
- [Helm Patterns](../kubernetes/helm-patterns.md)
- [Migrate to Flannel](../decisions/migrate-to-flannel.md)
```

- **YAML frontmatter** with `sources` linking back to contributing notes (temporal backlinks).
- **Standard markdown links** (`[Title](relative/path.md)`) for cross-references — works in all renderers.
- **`_index.md`** per topic folder: brief summary of every article in that topic.
- **`wiki/_index.md`** (master index): lists all topics with one-line descriptions. This is the LLM's entry point for navigation.
- The LLM **merges** new info into existing articles rather than creating duplicates.

## Slash Commands

### `/compile`

Incrementally compiles notes into wiki articles using git diff.

**Flow:**

1. Read `.wikime` to get last compiled commit hash. If missing, treat as first compile — process all notes.
2. Run `git diff <hash>..HEAD -- notes/` to identify changed note files.
3. Read changed notes in full (LLM needs full context, not just the diff lines).
4. Read `wiki/_index.md` to know existing topics and articles.
5. For each piece of new content, the LLM decides:
   - Belongs in an existing article → read it, merge new info, update `last_updated` and `sources`.
   - New topic/article → create folder, article, and update `_index.md` files.
   - New cross-links → add standard markdown links between related articles.
6. Write updated/new wiki files.
7. Update `wiki/_index.md` and relevant topic `_index.md` files.
8. Write current HEAD hash to `.wikime`.
9. Auto-commit the wiki changes.

### `/query`

Ask questions against the knowledge base. Returns markdown answers.

**Routing logic — the LLM infers from the question:**

- **Topical questions** ("what do I know about X?") → reads `wiki/_index.md`, navigates to relevant articles.
- **Temporal questions** ("what did I do this week?", "summarize last Monday") → reads notes directly.
- **Mixed questions** ("when did I last work on CNI stuff?") → uses wiki article `sources` frontmatter to bridge back to dated notes.

No configuration needed — the LLM naturally routes based on time words vs. topic words.

### `/lint`

Health check over the wiki. Scans for:

- **Broken links** — markdown links pointing to nonexistent files.
- **Orphan articles** — wiki articles with no links from other articles.
- **Stale articles** — `last_updated` is old but source notes have newer content.
- **Missing coverage** — topics mentioned frequently in notes but lacking a wiki article.
- **Duplicate concepts** — two articles covering the same thing under different names.

**Output:** Markdown report listing issues. Auto-fixes simple issues (broken links, missing index entries). Flags larger issues for human review.

## Design Properties

### Git-native

Everything is stored in the repo: notes, wiki, state (`.wikime`), and commands (Claude Code skills). No external database or state. This means:

- **Time travel** — checkout any historical commit and all commands work against that state.
- **Branching** — branch, fork, merge all work naturally.
- **Portable** — clone the repo anywhere with Claude Code and it works.

### Incremental compilation

Only processes notes that changed since last compile. The `.wikime` file stores the commit hash of the last successful compilation. This keeps compile fast even as the notes grow.

### Standard markdown

The wiki uses standard markdown with YAML frontmatter and relative markdown links. Works in any renderer (GitHub, VS Code, Obsidian, etc.). The repo can optionally include `.obsidian/` config for Obsidian users.

## Implementation

All three commands are implemented as Claude Code slash commands (skills). Each skill contains the prompt logic and any helper shell commands (e.g., git diff). Claude Code itself is the LLM engine — no separate API integration needed.

### Skills to create

1. **`/compile`** — the core compilation skill
2. **`/query`** — the Q&A skill
3. **`/lint`** — the health check skill

### No external dependencies

- No API keys to configure (uses Claude Code's session)
- No database (git is the database)
- No build tools (just markdown files)
- No package manager (just Claude Code skills + bash)
