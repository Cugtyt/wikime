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
