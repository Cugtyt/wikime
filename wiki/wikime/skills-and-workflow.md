---
title: Skills and Daily Workflow
topic: wikime
sources:
  - notes/2026/03-w5.md
last_updated: 2026-04-03
---

# Skills and Daily Workflow

wikime is an LLM-maintained personal knowledge base that turns weekly work notes into an organized topical wiki. Four slash-command skills power the system:

- **`/note`** — Quick-add entries to the current week's note file. Handles date sections and formatting automatically.
- **`/compile`** — Reads changed notes since the last compile, extracts topics, and creates or updates wiki articles. This is the core pipeline that keeps the wiki in sync with notes.
- **`/query`** — Ask natural-language questions against the knowledge base. Routes to wiki articles for topical questions and to notes for temporal questions.
- **`/lint`** — Health-checks the wiki for broken links, orphan articles, stale content, missing coverage, and duplicate concepts. Auto-fixes simple issues.

The intended workflow is daily: capture notes throughout the day with `/note`, then periodically `/compile` to keep the wiki current. `/lint` validates structural integrity after compiles.

## Key Takeaways

- All four skills were added and tested end-to-end as of 2026-04-03.
- The system is designed around a daily note-taking habit — consistency is the key input.
- `/compile` is incremental: it only processes notes that changed since the last compile, tracked via the `.wikime` marker file.

## Related

*No related articles yet.*
