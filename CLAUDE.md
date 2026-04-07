# wikime

LLM-maintained personal knowledge base. Weekly work notes in `notes/` are compiled into a topical wiki in `wiki/` via slash commands.

## Structure

- `notes/YYYY/MM-wN.md` — weekly notes (human-written, the only place humans write)
- `wiki/` — topical articles (LLM-maintained, never edit manually)
- `wiki/_index.md` — master index of all topics
- `wiki/<topic>/_index.md` — per-topic index
- `reports/` — generated weekly reports (LLM-generated, saved on request)
- `reviews/` — generated performance reviews (LLM-generated, saved on request)
- `.wikime` — last compiled commit hash

**Separation rule:** `notes/` contains only human-written weekly note files (`MM-wN.md`). All generated output goes to `wiki/`, `reports/`, or `reviews/`. Skills that read notes must only process files in `notes/`.

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

**The wiki is the impact memory.** Once a topic has impact context in the wiki (e.g. "cluster stability effort, goal: eliminate pod evictions, unblocking 3 teams"), new notes about that topic can inherit it automatically. Impact only needs to be described once per project/initiative.

**How each command handles impact:**

- `/note` — for each new significant bullet, check if the wiki has a related topic. If yes, propose the connection and inherited impact. If no, ask the user. Never re-ask about items from previous days. Only check new bullets.
- `/compile` — when synthesizing wiki articles, preserve and highlight impact context from notes prominently
- `/report` — flag significant items still missing impact context at end of week
- `/review` — flag gaps so the user can enrich before submitting their review

This is not optional polish — it's the core value of the system. Notes with impact context produce strong weekly reports and performance reviews. Notes without it produce generic summaries.

## Wiki Archiving

When the wiki grows large, topics that haven't been updated in a long time should be moved to `wiki/archive/`. This keeps the active wiki focused and fast to navigate.

- `wiki/archive/<topic>/` — same structure as active topics, just under archive
- `wiki/archive/_index.md` — index of all archived topics
- `wiki/_index.md` — only lists active topics, with a link to the archive index
- Archived articles are still searchable by `/query` and linkable from active articles
- `/compile` should suggest archiving when a topic's articles are all stale (no updates in 3+ months)
- `/lint` treats archive differently: only check broken links and index consistency. Skip stale articles, missing coverage, and duplicates — archived content is intentionally frozen

## Conventions

- All content is markdown
- Standard markdown relative links for cross-references (no wikilinks)
- Topic folders are lowercase kebab-case
- Article filenames are lowercase kebab-case
- _index.md files list all articles in their directory with one-line summaries
