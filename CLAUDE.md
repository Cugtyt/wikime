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

- Use standard markdown links for cross-references: `[Article Title](../topic/article-name.md)` (relative paths from the linking file)
- Each article has a `## Related` section with markdown links
- Each article has a `## Key Takeaways` section

## Commands

- `/note <what happened>` — quick-add to the current week's note file
- `/compile` — compile notes diff into wiki
- `/query <question>` — ask questions against the knowledge base
- `/lint` — health check the wiki
- `/standup` — generate yesterday/today/blockers from recent notes
- `/todos` — scan notes for open action items
- `/report [this week|last week]` — generate weekly report
- `/review [last 6 months|date range]` — generate performance review draft

## Impact Context

The primary goal of wikime is to capture not just what happened, but why it matters. Every significant piece of work should include impact context — who was affected, what improved, why it matters, or what it unblocked.

**Significant work:** fixing outages, launching features, unblocking teams, making decisions, completing migrations, setting up infrastructure, resolving incidents, onboarding people.

**Routine work** (attended standup, reviewed PR, read docs) does not need impact context.

**All commands that surface notes should flag impact gaps.** When a note item looks significant but lacks the "so what," nudge the user to add context. Examples:

- `/note` — after formatting bullets, scan the week's notes and flag missing impact before commit
- `/report` — include a "missing impact context" section for significant items
- `/review` — flag gaps so the user can enrich before submitting their review
- `/compile` — when synthesizing wiki articles, preserve and highlight impact context from notes

This is not optional polish — it's the core value of the system. Notes with impact context produce strong weekly reports and performance reviews. Notes without it produce generic summaries.

## Conventions

- All content is markdown
- Standard markdown relative links for cross-references (no wikilinks)
- Topic folders are lowercase kebab-case
- Article filenames are lowercase kebab-case
- _index.md files list all articles in their directory with one-line summaries
