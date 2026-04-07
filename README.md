# wikime

You do great work every day — fixing outages, shipping features, unblocking teams. But when it's time for standups, weekly reports, or performance reviews, you can't remember half of it.

wikime fixes this. Write quick notes as you work, and an LLM compiles them into an organized knowledge base that generates all those outputs for you.

## How it works

You write **temporal notes** — what happened today, this week:

```markdown
## Monday 2026-04-07
- Fixed Flannel MTU mismatch across staging — was causing pod evictions, unblocked 3 teams
- Talked to Sarah about PostgreSQL migration, targeting end of month
```

The LLM compiles them into a **topical wiki** — organized by subject, not date:

```markdown
---
title: CNI Debugging
topic: kubernetes
sources:
  - notes/2026/04-w1.md
last_updated: 2026-04-07
---

# CNI Debugging

Flannel pods in staging were crashing with OOMKilled errors, traced to
mismatched MTU settings. Fixed by setting `--iface=eth0` explicitly —
resolved cluster-wide networking instability, unblocking 3 dependent teams.

## Key Takeaways
- MTU mismatches between nodes are a common cause of CNI instability
- Explicitly setting `--iface` avoids interface ambiguity

## Related
- [Kubernetes Monitoring Stack](../onboarding/kubernetes-monitoring.md)
```

Raw notes go in, structured knowledge comes out. Git tracks everything.

## The key idea: impact tracking

Most note-taking tools capture *what* you did. wikime also captures *why it matters*.

When you write a note, the LLM checks the wiki for related topics. If it finds one, it proposes the connection: "This seems related to the cluster stability effort — unblocking 3 teams. Sound right?" If it's something new, it asks you for the impact.

You describe the impact once. The wiki remembers it. When you generate a performance review 6 months later, the impact is already there — no archaeology required.

## Commands

**Capture**

| Command | What it does |
|---------|-------------|
| `/note <text>` | Append to this week's note file. Formats into bullets, checks impact against wiki, asks to commit. |

**Organize**

| Command | What it does |
|---------|-------------|
| `/compile` | Diff notes since last compile, create/update wiki articles. Like a compiler transforms source to binary — this transforms raw notes into organized knowledge. |
| `/lint` | Health check: broken links, orphans, stale content, duplicates. Auto-fixes what it can. |

**Retrieve**

| Command | What it does |
|---------|-------------|
| `/query <question>` | Ask anything. Routes to wiki for topics, notes for dates, both for mixed queries. |
| `/standup` | Generate yesterday/today/blockers from recent notes. |
| `/todos` | Scan notes for open action items and infer completion status. |

**Reflect**

| Command | What it does |
|---------|-------------|
| `/report [week]` | Weekly report: accomplishments, decisions, blockers, next week. Flags impact gaps. |
| `/review [range]` | Performance review draft from months of notes. Frames work as impact, not activity. |

## Quick start

1. Clone this repo and open it in Claude Code
2. Start taking notes: `/note fixed the auth bug, unblocked the mobile team`
3. When ready: `/compile`

## A day with wikime

```
Morning:
  /standup
  > Yesterday: fixed Flannel MTU, unblocking 3 teams. Talked to Sarah about PG migration.
  > Today: No plan yet — use /note to add one.
  > Blockers: need rollback plan for migration

Throughout the day:
  /note set up HPA for auth service to handle traffic spikes after launch
  > This connects to "Auth Service Migration" — targeting end of month. Sound right?
  > yes
  > Commit? yes

End of week:
  /compile
  /report
  > Accomplishments: resolved cluster-wide networking, HPA for auth service...
  > Items missing impact: "reviewed Jake's PR" — (skipped, routine)

Every 6 months:
  /review last 6 months
  > Impact Summary:
  > - Resolved cluster-wide networking instability, unblocking 3 dependent teams
  > - Led PostgreSQL migration, completed on schedule
  > - Established Kubernetes monitoring stack, onboarded 2 team members
```

Your notes accumulate, the wiki grows, and your knowledge compounds.

## Repo structure

```
notes/                    ← you write here (via /note or manually)
  2026/
    04-w1.md              ← weekly note
wiki/                     ← LLM maintains this
  _index.md               ← master index
  kubernetes/
    cni-debugging.md
  python/
    async-patterns.md
.wikime                   ← last compiled commit hash
.claude/skills/           ← the eight commands
```

Inspired by [Andrej Karpathy's idea](https://x.com/karpathy) of using LLMs to build and maintain personal knowledge bases.
