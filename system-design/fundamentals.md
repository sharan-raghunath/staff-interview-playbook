# System Design Fundamentals

Architecture is about assigning responsibilities and making trade-offs.


## Guided design process

Use `interview-process.md` for requirement gathering, summaries, assumptions, responsibility decomposition, deep dives and wrap-up.


## Building blocks covered

- [Load Balancers](load-balancers.md): motivation, routing, health checks, L4/L7 and sticky sessions.


## Caching

Caching improves latency and source-of-truth scalability by reusing data, while introducing bounded staleness and invalidation concerns.

Key learned dimensions:

- cache-aside versus read-through responsibility;
- database-first invalidation versus cache update/write-through/write-behind;
- in-memory L1 versus distributed L2;
- TTL, LRU/LFU, warming and stampede;
- data-specific cache/no-cache decisions.

See [`caching.md`](caching.md).

## Resilience: retries and circuit breakers

Retries handle transient failures. Circuit breakers prevent persistent downstream failure from cascading. Long-running durable work should use delayed queue retries, and recovered services should receive backlog through controlled ramp-up rather than an immediate flood.

See [`../backend/circuit-breakers.md`](../backend/circuit-breakers.md).
