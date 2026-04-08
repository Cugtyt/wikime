---
title: Auth Service Migration
topic: postgresql-migration
sources:
  - notes/2026/04-w15.md
last_updated: 2026-04-08
---

# Auth Service Migration

Migrating the auth service from Redis to PostgreSQL to address a compliance requirement — legal flagged the old session token storage format. Team decided to proceed on April 8, targeting end of April. Sarah is leading, Jake supporting.

## Problem

The auth service stores session tokens in Redis in a format that doesn't meet new compliance requirements flagged by legal. The token format needs to change, and the storage backend needs to support the new schema.

## Decision

Team approved the migration on 2026-04-08. Key factors:
- Compliance is the primary driver — not optional
- PostgreSQL gives us better schema enforcement for the new token format
- Timeline: end of April
- Main risk: session table schema change needs a rollback plan

## Open Items

- Write rollback plan before starting migration
- Finalize session table schema design
- Test token format migration on staging

## Key Takeaways

- Compliance-driven migrations have non-negotiable deadlines — plan accordingly
- Always have a rollback plan before starting a data migration

## Related

None yet — relationships will emerge as the migration progresses.
