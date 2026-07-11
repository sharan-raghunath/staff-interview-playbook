# Changelog

## Day 13

- Added Day 13 journal.
- Added `backend/async-await-internals.md`.
- Added `coding/longest-substring-without-repeating-characters.md`.
- Added Variable-size Sliding Window to coding patterns.
- Updated README progress, roadmap and pending items.
- Recorded that system design and mock interview were explicitly deferred.

## Day 11

- Added Day 11 journal.
- Added `coding/merge-two-sorted-lists.md`.
- Added `system-design/idempotent-consumer-inbox.md`.
- Added ADR-014 for normalizing job stage state.
- Added ADR-015 for idempotent consumer strategy.
- Updated Invoice Manager docs with `Jobs` + `JobStages` target schema decision.
- Updated roadmap, context, scorecard, vocabulary, coding patterns and pending items.


## Day 1

### Coding
- Two Sum
- Hash Lookup pattern

### Backend
- Why APIs exist

### System Design
- URL Shortener introduction

---

## Day 2

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

## Day 3

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

## Day 4

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

## Day 5

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

## Day 6

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

## Day 7

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

## Day 8

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
- Added comprehensive Day 8 journal
- Added queue fundamentals as a canonical document
- Added failure-propagation design note
- Added linked-list-cycle coding note
- Added ADR-008 and ADR-009
- Updated context, roadmap, pending items, scorecard and skills matrix
- Removed premature queue-topic leakage from learned-content documents


---

## Day 9

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
- Added comprehensive Day 9 journal.
- Added coding notes for LeetCode 876.
- Updated queue fundamentals, Invoice Manager target-state notes, context, roadmap, pending list, scorecard, README, and start-here context.
- Removed future-only terms from learned vocabulary.

---

## Day 10

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

## Day 12

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
- Added Day 12 journal, coding note and backpressure note.
- Updated context, roadmap, pending topics, skills matrix, scorecard, vocabulary, README and start-here context.
- Added explicit “mentioned but not covered” status for KEDA, Gunicorn and global concurrency.
