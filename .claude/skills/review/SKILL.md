---
name: review
description: Generate a performance review draft from accumulated notes. Frames work as impact, not activity. Can save to file if requested. Usage - /review [last 6 months | 2026 H1 | 2025-10 to 2026-03]
argument-hint: [last 6 months | 2026 H1 | date range]
allowed-tools: Read, Write, Glob, Grep, Bash(mkdir*)
---

# Performance Review

Generate a performance review draft from your notes and wiki.

## Context

Today:
!`date +%Y-%m-%d`

Available notes:
!`find notes/ -name "*.md" 2>/dev/null | sort`

Available wiki:
!`find wiki/ -name "*.md" 2>/dev/null | sort`

**Time range:** $ARGUMENTS

If no argument given, default to "last 6 months." Parse the time range to determine which note files to read.

## Steps

1. **Read all note files in the time range.** If no matching notes exist, respond: "No notes found for that period. A review needs at least a few weeks of notes." and stop.

2. **Read `wiki/_index.md`** and relevant wiki articles for synthesized context.

3. **Read any saved weekly reports** from `reports/` in the range if they exist.

4. **Generate the review** in the format below.

## Review Format

```markdown
# Performance Review — [date range]

## Impact Summary
Top 3-5 most significant contributions. Frame as outcomes:
- "Resolved cluster-wide networking instability by identifying MTU mismatch, eliminating pod evictions and unblocking 3 teams"
- NOT "fixed Flannel MTU issue"

## Key Projects
For each major project/workstream:
### [Project Name]
- **What:** What you did and the decisions you drove
- **Impact:** Outcome, who benefited, scope
- **Collaboration:** Who you worked with, cross-team coordination

## Technical Growth
- New skills, tools, domains, or technologies learned
- Progression in depth or breadth

## Collaboration & Leadership
- Cross-team work and coordination
- Mentoring and onboarding (e.g. "onboarded Jake to monitoring stack")
- Decisions you drove or influenced
- Code reviews, design reviews, knowledge sharing

## Challenges Overcome
- Hard problems debugged, incidents resolved
- Blockers you unblocked for yourself or others
- Shows resilience and problem-solving ability

## Metrics & Evidence
- Any concrete numbers, timelines, or measurable outcomes from notes
- "Completed migration 1 week ahead of schedule"
- "Reduced staging incidents from X to 0"
```

## Key Principles

- **Impact over activity.** Always answer "so what?" Every bullet should state the outcome, not just the action.
- **Use the strongest framing the evidence supports.** Don't fabricate, but don't undersell either. If the notes say "fixed MTU, unblocked 3 teams" then say "resolved cluster-wide issue, unblocking 3 dependent teams."
- **Group by theme, not by date.** A performance review tells the story of your impact, not a chronological log.
- **Name names.** Collaboration is more credible with specifics: "worked with Sarah on migration plan" not "collaborated with teammates."

## Gap Analysis

After the review, follow the Impact Context rules in `CLAUDE.md`. Flag significant items missing the "so what" so the user can enrich their notes before submitting. Suggest using `/note` to add the missing context to the relevant week's file.

## Output

Output to terminal. Do not save to file unless the user asks.

After presenting, suggest: "Want me to save this draft to `reviews/YYYY-HN.md`?"
