---
title: Auth Service PostgreSQL Migration
topic: postgresql-migration
sources:
  - notes/2026/04-w1.md
last_updated: 2026-04-03
---

# Auth Service PostgreSQL Migration

The auth service is being migrated to PostgreSQL, targeting end of April 2026. The migration is driven by both technical needs and a compliance requirement around session token storage.

## Background

Sarah and the team agreed to proceed with the migration. Legal flagged the old session token format as non-compliant, making this migration a compliance priority rather than just a technical improvement.

## Key Concerns

- **Session table schema** — the main technical concern during migration planning
- **Session token format change** — required for compliance; the old format was flagged by legal

## Timeline

- Decision made: 2026-04-03
- Target completion: end of April 2026

## Key Takeaways

- Migration is compliance-driven (legal flagged old session token format)
- Session table schema design is the primary technical risk
- Team alignment achieved — Sarah and others are in favor

## Related

- [[kubernetes/cni-debugging]] — related infrastructure work happening in parallel
