# Session 18 Journal

**Date:** 18 July 2026  
**Status:** Complete

## Backend Theory — Circuit Breakers

Covered:

- cascading failure and retry amplification;
- Closed, Open and Half-Open states;
- retries for transient failures versus circuit breakers for persistent failures;
- failure classification, including why `400` and business failures neither retry nor increment the breaker;
- exponential backoff and jitter;
- delayed queue retry for long-running OCR/GPU work;
- recovery surge / thundering herd;
- controlled ramp-up, bounded concurrency and rate limiting after recovery.

Adaptive rate limiting and adaptive concurrency were briefly introduced when asked, but were new concepts and are deliberately deferred to a dedicated future lesson.

## System Design Revision

Revised and connected the following Invoice Manager reliability concepts:

- idempotency keys and database uniqueness;
- Correlation ID vs Message ID vs Idempotency Key;
- transactional Outbox;
- Inbox / idempotent consumer;
- deterministic Blob paths;
- visibility timeout and leases;
- retries and reconciliation after crashes.

No new architecture decision was added; this was consolidation of already learned design.

## Coding — Daily Temperatures

Derived the monotonic-stack mechanism from first principles rather than naming the pattern first:

1. Started with O(n²) nested-loop brute force.
2. Identified unresolved elements waiting for a future greater value.
3. Chose a stack because the most recent unresolved item should be checked first.
4. Derived the descending/non-increasing invariant from the pop behavior.
5. Recognized that Daily Temperatures must store indices, compare values at those indices and calculate `currentIndex - poppedIndex`.
6. Implemented the O(n) C# solution independently.

Minor cleanup: the final loop assigning zeroes was unnecessary because `int[]` is zero-initialized.

## Coaching Correction

New DSA patterns should be discovered in this order:

1. brute force;
2. identify repeated work;
3. identify unresolved information;
4. derive the data structure;
5. derive the invariant;
6. analyze complexity;
7. implement;
8. name the pattern.

This guided-discovery method is now the preferred default.

## Outcome

- Backend theory complete.
- System-design revision complete.
- Daily Temperatures complete.
- Session 18 complete.
- Resume with Session 19 on Monday.
