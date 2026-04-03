---
title: Async Patterns
topic: python
sources:
  - notes/2026/04-w1.md
last_updated: 2026-04-03
---

# Async Patterns

Python's `asyncio` library provides structured concurrency primitives for managing concurrent operations cleanly.

## asyncio.TaskGroup

`TaskGroup` enables structured concurrency — a pattern where a group of tasks is managed as a unit. All tasks in the group must complete (or fail) before execution continues past the group's context manager.

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(fetch_users())
    tg.create_task(fetch_orders())
# Both tasks guaranteed to be done here
```

This avoids the pitfalls of fire-and-forget `create_task()` calls where exceptions can be silently lost.

## asyncio.Semaphore

`Semaphore` limits concurrent access to a resource — useful for rate limiting outbound API calls.

```python
sem = asyncio.Semaphore(10)  # max 10 concurrent

async def rate_limited_fetch(url):
    async with sem:
        return await fetch(url)
```

## Key Takeaways

- Use `TaskGroup` for structured concurrency instead of bare `create_task()`
- Use `Semaphore` to cap concurrent API calls and avoid overwhelming external services
- Both patterns make async code more predictable and debuggable

## Related

- [[postgresql-migration/auth-service-migration]] — async patterns may be relevant for migration tooling
