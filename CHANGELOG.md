# Changelog

## Session 17

### Backend

- Added race conditions, critical sections, mutual exclusion and basic C# `lock`.
- Added `backend/thread-safety-and-lock.md`.
- Kept advanced synchronization and distributed locking pending.

### Coding

- Added LeetCode 150 Evaluate Reverse Polish Notation.
- Added postfix stack-evaluation pattern and explicit production stack examples.

### System Design

- Added comprehensive caching fundamentals and Invoice Manager strategy.
- Covered cache-aside, read-through, write-through/write-behind, invalidation, stampede, L1/L2, TTL, LRU/LFU, warming and conceptual cross-pod invalidation.
- Recorded selective caching for tenant/reference data and no-cache decisions for OCR payloads, final extraction results by default, and uploaded PDFs.

### Repository

- Added `session-journal/session-017.md`.
- Reconciled README, START_HERE, context, roadmap, pending items, skills matrix, scorecard, coding patterns, system-design fundamentals and interview mistakes.

## Session 16

- Added `ValueTask<T>` single-consumption usage guidance while keeping advanced internals pending.
- Added Load Balancer fundamentals covering motivation, routing strategies, health checks, Layer 4 versus Layer 7, sticky sessions and failover implications.
- Added LeetCode 155 Min Stack and the stack-with-auxiliary-state pattern.
- Added `session-journal/session-016.md`.
- Updated README, start context, curriculum, roadmap, pending work, skills matrix and scorecard.
- Added the rule that sessions may span calendar days and all progress references must remain session-based.

## Session 15

### Backend

- Added `Task<T>` versus `ValueTask<T>` fundamentals.
- Recorded hot-path, synchronous-completion and profiling criteria.
- Kept advanced `ValueTask<T>` topics explicitly pending.

### Coding

- Added Stack/LIFO fundamentals.
- Added Valid Parentheses with an optimal O(n) solution.
- Recorded C# review improvements: `TryGetValue`, `TryPop`, and `Count == 0`.

### Curriculum and Repository

- Updated normal weekday cadence to backend theory + coding + one small system-design building block + journal reconciliation.
- Retained Friday as interview-simulation and weekly-review day.
- Reconciled README, START_HERE, context, roadmap, pending topics, skills matrix, scorecard and coding patterns.

## Session 14

- Migrated journal and curriculum terminology from Day to Session.
- Renamed `daily-journal/` to `session-journal/` and historical journal files accordingly.
- Added `CURRICULUM.md` as the coaching contract and repository authority.
- Added reusable `system-design/interview-process.md`.
- Added guided `system-design/notification-service.md`.
- Added `coding/longest-repeating-character-replacement.md`, accurately marked as excessively guided.
- Added Session 14 journal.
- Clarified Invoice Manager API Service as owner of the final mapped business state and completion notification event.
- Recorded Friday as preferred simulation day and weekends as optional.
- Kept detailed DR and circuit breaker pending rather than learned.
- Updated README, START_HERE, context, roadmap, pending list, skills matrix and scorecard.

## Session 13

- Added Session 13 journal.
- Added `backend/async-await-internals.md`.
- Added `coding/longest-substring-without-repeating-characters.md`.
- Added Variable-size Sliding Window to coding patterns.
- Updated README progress, roadmap and pending items.
- Recorded that system design and mock interview were explicitly deferred.

## Session 11

- Added Session 11 journal.
- Added `coding/merge-two-sorted-lists.md`.
- Added `system-design/idempotent-consumer-inbox.md`.
- Added ADR-014 for normalizing job stage state.
- Added ADR-015 for idempotent consumer strategy.
- Updated Invoice Manager docs with `Jobs` + `JobStages` target schema decision.
- Updated roadmap, context, scorecard, vocabulary, coding patterns and pending items.


## Session 1

### Coding
- Two Sum
- Hash Lookup pattern

### Backend
- Why APIs exist

### System Design
- URL Shortener introduction

---

## Session 2

### Coding
- Best Time to Buy and Sell Stock
- Running Minimum pattern

### Backend
- ASP.NET Core Request Pipeline
- JWT validation introduction
- Zero Trust

### Production Architecture
- Initial API Service vs Orchestrator discussion

---

## Session 3

### Coding
- Valid Anagram
- Frequency Counting pattern

### Backend
- Stack vs Heap introduction
- Arrays as reference types
- Character arithmetic (`c - 'a'`)

### Production Architecture
- Text Extractor vs Data Extractor
- Gateway responsibilities
- Responsibilities vs Components

---

## Session 4

### Coding
- Product of Array Except Self
- Prefix / Suffix Computation
- Space optimization by reusing the output array

### Backend
- Why HTTP is stateless
- Why Sessions were invented
- How Cookies transport Session IDs
- Browser vs Angular vs ASP.NET responsibilities

### Production Architecture
- Synchronous vs asynchronous communication introduction
- Current Invoice Manager synchronous bottlenecks

---

## Session 5

### Backend
- Distributed Sessions using Redis
- JWT fundamentals, JWKS and key rotation
- Claims, RBAC/ABAC, roles/scopes
- Access/refresh token behavior and revocation trade-offs

### Coding
- Valid Palindrome
- Two Pointers pattern

### System Design
- Current fail-fast behavior and need for recovery motivation

---

## Session 6

### Backend / Security
- OAuth 2.0 and OIDC from first principles
- Authorization Code, Implicit, Client Credentials and OBO concepts
- Token lifecycle, logout, SSO/SLO and Zero Trust identity principles

### Coding
- Maximum Average Subarray I
- Fixed-size Sliding Window

### System Design
- Async API contract using `202 Accepted`, `jobId` and `statusUrl`
- Idempotency
- Correlation ID vs Idempotency Key
- Retry and DLQ introduction

---

## Session 7

### Rapid-Fire Interview
- Browser-to-Invoice-Manager request lifecycle
- Current vs future failure handling
- Zero Trust and compromised-service blast radius
- Invoice Manager data ownership

### Architecture / Documentation
- Reviewed proprietary architecture relationships without reproducing visuals
- Added client edge path, identity federation path and platform/application boundaries
- Added Redis, Azure CV and Azure DI context

### System Design
- Async Invoice Manager redesign Part 1
- Queue placement discussion
- SQL selected as target-state processing-state authority
- OCR artifact persistence rationale

### Coding
- Find Pivot Index
- Prefix Sum / Running Sum

---

## Session 8

### System Design — Queue Fundamentals
- Derived why long-running business operations must outlive HTTP requests
- Added Job ID and durable SQL job-state model
- Derived queue as durable producer/consumer handoff
- Derived temporary message ownership and visibility timeout
- Derived ownership renewal for long-running OCR
- Clarified Orchestrator owns queue interaction in the incremental target state; TE stays queue-unaware while invoked over HTTP
- Derived stage completion rule and deferred transactional atomicity problem
- Classified business validation, retryable dependency and operational failures
- Added exponential backoff concept and maximum retry count
- Defined DLQ policy and controlled batch replay
- Defined user-facing vs technical failure representation

### Coding
- Linked List Cycle
- Fast & Slow Pointers / Floyd’s cycle detection
- HashSet brute force: O(n) time, O(n) space
- Optimized pointers: O(n) time, O(1) space

### Repository
- Added comprehensive Session 8 journal
- Added queue fundamentals as a canonical document
- Added failure-propagation design note
- Added linked-list-cycle coding note
- Added ADR-008 and ADR-009
- Updated context, roadmap, pending items, scorecard and skills matrix
- Removed premature queue-topic leakage from learned-content documents


---

## Session 9

### System Design — Queue Settlement and Recovery
- Distinguished ownership renewal, release-for-retry, and successful message completion.
- Derived durable-success-before-completion rule.
- Derived ambiguous acknowledgement recovery and redelivery reconciliation.
- Added competing consumers, lease expiry, atomic SQL stage claim, and per-job ordering.
- Clarified scaling responsibility across Orchestrator, TE, and external OCR dependencies.

### Azure Service Bus
- Mapped learned model to Peek-Lock, message locks, lock renewal, Complete, Abandon/expiry, and DLQ.
- Covered queue versus topic for the current scope.
- Deliberately did not select Service Bus sessions.

### Invoice Manager Target Topology
- Selected separate OCR and field-extraction queues.
- Defined OCR → durable artifact/SQL state → field extraction progression.
- Added browser polling through IM Web/Edge from authoritative SQL state.
- Recorded Billing boundary after user-reviewed submission; Orchestrator does not own Billing.
- Added ADR-010 and ADR-011.

### Coding
- Middle of the Linked List.
- Fast & Slow Pointers used to locate the second middle node in O(n) time and O(1) space.

### Repository
- Added comprehensive Session 9 journal.
- Added coding notes for LeetCode 876.
- Updated queue fundamentals, Invoice Manager target-state notes, context, roadmap, pending list, scorecard, README, and start-here context.
- Removed future-only terms from learned vocabulary.

---

## Session 10

### Coding
- Reverse Linked List.
- Iterative pointer manipulation using `prev`, `curr`, and `next`.
- Recursive reverse linked list for interview readiness.
- Base case: `head == null || head.next == null`.
- Recursive unwinding: `head.next.next = head` and `head.next = null`.

### Distributed Systems
- Identified the SQL update + queue publish reliability gap.
- Learned the Transactional Outbox pattern.
- Covered outbox row lifecycle: pending, published, retained, cleaned up later.
- Covered at-least-once publishing and duplicate publish risk.
- Connected outbox duplicates to idempotent consumer design and atomic SQL stage claims.

### Invoice Manager
- Expanded target pipeline to include PDF preparation / page-image generation before OCR.
- Applied the durable-output-before-stage-completion rule to:
  - original PDF upload;
  - PDF preparation artifacts;
  - OCR artifacts;
  - field-extraction prediction output;
  - user submit → billing request.
- Added ADR-012 for Transactional Outbox stage handoffs.
- Added ADR-013 for marking stages complete only after durable output exists.


---

## Session 12

### Distributed Systems / Backend
- Learned backpressure and bottleneck-oriented scaling.
- Distinguished internal service saturation from external-provider rate limiting.
- Introduced per-pod bounded concurrency conceptually through `SemaphoreSlim`.
- Deferred strict global concurrency and distributed rate limiting.
- Recorded TE sync and DE `gthread` production choices without marking Gunicorn internals as covered.

### Coding
- Completed Remove Nth Node From End of List.
- Derived two-pass brute force and one-pass fixed-gap pointer solution.
- Reinforced dummy-head handling for head deletion.
- Clarified that one pass does not require a single loop.

### Friday Mock
- Investigated a delayed 2,500-page document using Job ID and Correlation ID.
- Practiced stage timeline analysis and metrics → traces → logs.
- Reinforced artifact/state reconciliation when Blob output exists but SQL remains `Processing`.

### Repository
- Added Session 12 journal, coding note and backpressure note.
- Updated context, roadmap, pending topics, skills matrix, scorecard, vocabulary, README and start-here context.
- Added explicit “mentioned but not covered” status for KEDA, Gunicorn and global concurrency.

## Session 18

### Backend / Distributed Systems
- Added circuit breaker fundamentals: Closed, Open and Half-Open.
- Distinguished transient retries from persistent-failure protection.
- Added failure classification, retry amplification, backoff/jitter and delayed queue retry.
- Added recovery-surge/thundering-herd guidance and controlled ramp-up.
- Kept adaptive concurrency and adaptive rate limiting pending.

### Coding
- Added Daily Temperatures.
- Derived the introductory monotonic-stack invariant from brute force.
- Stored unresolved indices and completed an O(n) C# implementation.

### Repository
- Added Session 18 journal and durable circuit-breaker/Daily Temperatures notes.
- Updated curriculum, README, start-here, context, roadmap, pending list, skills matrix, scorecard, mistakes and pattern references.
