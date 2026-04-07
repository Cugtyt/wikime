# wikime

LLM-maintained personal knowledge base. You write weekly work notes, Claude Code compiles them into a topical wiki.

Inspired by Andrej Karpathy's idea of using LLMs to build and maintain personal knowledge bases.

## How it works

```
You write notes → /compile → Wiki auto-generated → /query to search
     ↑                                                    |
     └────────── knowledge accumulates over time ─────────┘
```

Raw weekly notes in `notes/` are the input. The LLM reads your notes, extracts topics, synthesizes articles, maintains cross-links and indexes — all in `wiki/`. You rarely touch the wiki directly. Git tracks everything.

## Commands

| Command | What it does |
|---------|-------------|
| `/note <text>` | Append to current week's note file. Auto-detects week, formats into bullets, asks to commit. |
| `/compile` | Git diff since last compile, update wiki articles incrementally. Asks for confirmation only when decisions are ambiguous. |
| `/query <question>` | Ask anything. Routes to wiki for topics, notes for dates, both for mixed queries. |
| `/lint` | Health check: broken links, orphan articles, stale content, missing coverage, duplicates. Auto-fixes what it can. |
| `/standup` | Generate yesterday/today/blockers from recent notes. |
| `/todos` | Scan notes for open action items and infer completion status. |
| `/report [week]` | Generate weekly report: accomplishments, decisions, blockers, next week. |

## Repo structure

```
notes/                    ← you write here (via /note or manually)
  2026/
    04-w1.md              ← weekly note
    04-w2.md
wiki/                     ← LLM maintains this
  _index.md               ← master index
  kubernetes/
    _index.md
    cni-debugging.md
  python/
    _index.md
    async-patterns.md
.wikime                   ← last compiled commit hash
.claude/skills/           ← the seven commands above
```

## Quick start

1. Clone this repo and open it in Claude Code
2. Start taking notes: `/note fixed the auth bug, talked to Sarah about the migration timeline`
3. When ready, compile: `/compile`

## Workflow

```
/note debugged k8s pod eviction issue, was caused by memory limits on the sidecar container
> commit? yes

/note sarah confirmed postgres migration for next sprint, need to write rollback plan

/compile
> wiki/kubernetes/pod-eviction.md created
> wiki/decisions/postgres-migration.md updated

/query what do I know about pod eviction?
> Based on wiki/kubernetes/pod-eviction.md: ...

/lint
> All clear. 0 issues found.

/standup
> Yesterday: debugged pod eviction, confirmed postgres migration
> Today: No plan logged yet — use /note to add one.
> Blockers: need to write rollback plan

/todos
> Open: write rollback plan (Apr 3), follow up with Sarah on timeline (Apr 3)
> Done: fix Flannel MTU (Apr 1 → resolved Apr 2)

/report
> Accomplishments: resolved k8s pod eviction, postgres migration approved...
```

Your notes accumulate, the wiki grows, and your knowledge compounds.
