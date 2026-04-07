---
name: lint
description: Health check the wikime knowledge base. Finds broken links, orphan articles, stale content, missing coverage, and duplicate concepts. Auto-fixes simple issues.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git add*), Bash(git commit*)
---

# Lint the Knowledge Base

You are the wikime linter. Scan the wiki for quality issues and fix what you can.

## Available data

Active wiki articles:
!`find wiki/ -path wiki/archive -prune -o -name "*.md" -print 2>/dev/null | sort || echo "No wiki found — run /compile first"`

Archived wiki articles:
!`find wiki/archive/ -name "*.md" 2>/dev/null | sort || echo "No archive"`

Notes files:
!`find notes/ -name "*.md" 2>/dev/null | sort || echo "No notes found"`

## Checks to perform

Read all wiki markdown files, then check for the following. **Important:** for archived content (`wiki/archive/`), only run checks 1 (Broken Links) and 6 (Index Consistency). Skip checks 2-5 — archived content is intentionally frozen. See CLAUDE.md for archiving rules.

### 1. Broken Links
Scan all articles for markdown links `[text](path.md)`. For each relative link, verify the target file exists (resolving the path relative to the linking file). Report any broken links.

**Auto-fix:** Remove broken links from `## Related` sections.

### 2. Orphan Articles
Find wiki articles (excluding `_index.md` files) that are not linked from any other article's markdown links or `_index.md`. Every article should be reachable from the master index.

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

## Broken Links
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

## Action

After presenting the report, decide what to do:

**Straightforward auto-fixes** (broken links, missing index entries, index consistency) — apply immediately without asking. These are mechanical and safe.

**Judgment calls** (merging duplicates, creating articles for missing coverage, resolving stale content) — present your recommendation and wait for user input before acting. Example:
```
Recommended actions:
- AUTO-FIX: Added 2 missing entries to wiki/kubernetes/_index.md
- QUESTION: wiki/python/async-patterns.md and wiki/python/concurrency.md look like duplicates — merge them? If so, which title do you prefer?
- SUGGESTION: "grafana" appears 4 times in notes — want me to run /compile to create an article?
```

If no judgment calls are needed and all fixes are mechanical, just apply them and report what you did.

If any fixes were applied, stage and commit:
```bash
git add wiki/
git commit -m "wikime: lint — auto-fixed N issues"
```
