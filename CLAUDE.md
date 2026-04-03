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

## Conventions

- All content is markdown
- Standard markdown relative links for cross-references (no wikilinks)
- Topic folders are lowercase kebab-case
- Article filenames are lowercase kebab-case
- _index.md files list all articles in their directory with one-line summaries
