---
title: Async Patterns
topic: python
sources:
  - notes/2026/04-w15.md
last_updated: 2026-04-08
---

# Async Patterns

Practical asyncio patterns for concurrent Python services — `TaskGroup` for structured concurrency and `Semaphore` for rate limiting external API calls. Both are relevant to the batch processing service and the new rate limiter.

## TaskGroup — structured concurrency

`asyncio.TaskGroup` (Python 3.11+) manages a group of concurrent tasks with automatic cleanup. If any task fails, all others are cancelled — no orphaned coroutines.

Useful for: the batch processing service, where we need to process multiple items concurrently but want clean failure handling.

## Semaphore — rate limiting

`asyncio.Semaphore` limits the number of concurrent operations. Use it to rate-limit calls to external APIs that have request caps.

Useful for: the new rate limiter for external API calls.

## Key Takeaways

- `TaskGroup` replaces manual `gather()` + error handling — cleaner and safer
- `Semaphore` is the simplest way to add concurrency limits without a full rate-limiting library
- Both are stdlib — no external dependencies needed
