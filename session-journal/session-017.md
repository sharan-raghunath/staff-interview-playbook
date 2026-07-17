# Session 17 — Thread Safety, Reverse Polish Notation and Caching

## Scope

- Backend theory: race conditions, critical sections, mutual exclusion and basic C# `lock` usage.
- Coding: LeetCode 150 — Evaluate Reverse Polish Notation.
- System-design building block: caching fundamentals through an Invoice Manager-specific strategy.

The session remained guided learning. It spanned multiple calendar periods and is tracked only as Session 17.

## Backend — Thread Safety and `lock`

A shared counter demonstrated why `Counter++` is not one atomic operation. It is a read-modify-write sequence, and two threads can read the same old value and overwrite one another's increments.

Learned:

- race conditions arise when results depend on the interleaving of concurrent access to shared mutable state;
- protecting only the write or only the modification is insufficient;
- the entire read-modify-write sequence is the critical section;
- mutual exclusion allows only one thread to execute that section at a time;
- C# `lock` provides basic mutual exclusion when all callers use the same lock object;
- lock objects should be `private readonly` so synchronization remains class-owned and the reference does not change.

Intentionally deferred:

- lock re-entrancy internals;
- deadlocks;
- `Monitor`, `Interlocked`, `SemaphoreSlim` and reader/writer locks;
- distributed locking.

## Coding — LeetCode 150: Evaluate Reverse Polish Notation

### Clarifications

- integer division truncates toward zero;
- input is valid;
- divide-by-zero does not occur;
- every operator has two operands;
- the expression returns one integer.

### Approach

The stack solution was identified directly because postfix notation naturally consumes the latest unfinished operands first.

- push integers;
- on an operator, pop the right operand, then the left operand;
- calculate `left operator right`;
- push the intermediate result;
- return the final stack value.

The user implemented the correct O(n) solution in C# and deliberately avoided defensive hooks that the valid-input contract did not require.

### Complexity

```text
Time:  O(n)
Space: O(n)
```

### Production connection

Explicit `Stack<T>` uses discussed included undo/redo, iterative DFS, parser or expression state, reverse-order compensation actions and navigation/backtracking. A literal stack is less common in CRUD-heavy services than queues, lists or dictionaries.

## System Design — Caching

### Fundamentals

Caching reduces repeated source-of-truth reads, latency and database load, but introduces stale-read risk. The freshness requirement must be defined before selecting a TTL or invalidation strategy.

### Patterns covered

- **Cache-aside:** preferred default for the current enterprise architecture because the application retains authorization, validation, tenant and mapping responsibilities.
- **Read-through:** cache loads the source on a miss; potentially expands coupling, credentials, trust boundary and attack surface.
- **Database first then invalidate:** preferred ordinary write path.
- **Write-through:** cache synchronously persists before acknowledgement.
- **Write-behind:** cache acknowledges before later persistence; fast but requires durable pending state and reconciliation.

### Concurrency and topology

- concurrent misses for one key can create a cache stampede;
- per-key coordination avoids serializing unrelated keys;
- an in-process lock does not coordinate multiple pods;
- L1 is fastest but local to a pod;
- L2 is shared but requires a network hop;
- multi-level reads use L1 → L2 → source of truth;
- L1 generally needs a shorter TTL than L2 because local invalidation is not automatically cross-pod.

### TTL, eviction and warming

- TTL follows volatility, stale-data impact, propagation expectations and rebuild cost;
- LRU favors recent use;
- LFU favors frequent use but pure historical counts can penalize newly hot data;
- cache warming should preload only known or measured hot data;
- rare changes to pod-local reference caches can be broadcast for invalidation rather than triggering rolling restarts.

### Invoice Manager decisions

- **Tenant configuration:** cache, split by volatility/freshness, use explicit invalidation for admin changes.
- **OCR payload:** do not cache; large, low reuse and already durable in Blob storage.
- **Final extraction result:** do not rerun GPT on refresh; read persisted versioned output. No cache by default because reuse is low.
- **Reference data:** small static sets can use long-lived L1 entries with warming and explicit invalidation.
- **Uploaded PDFs:** do not cache; large, low reuse, PII risk and Blob storage is the correct durable store.

## Interview and Coaching Notes

- Trust the explicit coding contract and avoid defensive over-engineering when invalid input is ruled out.
- Cache decisions must be data-specific; “cache frequently used data” is not enough.
- Security and attack-surface effects belong in caching pattern trade-offs.
- Do not label unsupported historical comparisons as personal improvement.

## Session Status

Completed:

- thread-safety fundamentals and basic `lock` usage;
- Evaluate Reverse Polish Notation;
- caching fundamentals, advanced introductory patterns and Invoice Manager application;
- durable repository reconciliation.

Pending for future sessions:

- advanced .NET synchronization;
- distributed locking;
- Redis internals and cache sizing;
- detailed cache-coherence and invalidation mechanisms;
- formal negative caching/cache-penetration/cache-avalanche coverage;
- Friday interview simulation according to curriculum scope.
