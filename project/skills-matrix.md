# Skills Matrix

This is a living assessment based on completed sessions.

## Backend / Security

| Area | Current Level | Evidence |
|---|---:|---|
| HTTP Statelessness | 9/10 | Can explain why sessions were invented |
| Sessions and Cookies | 9/10 | Can explain session ID vs server-side session data |
| Distributed Sessions | 8.5/10 | Understands Redis/shared-store trade-offs |
| JWT | 9/10 | Can explain signature, JWKS, `kid`, claims and revocation trade-offs |
| OAuth 2.0 | 8.5/10 | Can derive delegation, auth code, client credentials and OBO concept |
| OIDC | 8.5/10 | Can distinguish ID token from access token |
| Zero Trust Identity | 9/10 | Can reason about service identity vs user identity |
| Async/Await Internals | 9/10 | Can explain state machine, MoveNext, context capture and scheduling |
| ValueTask Fundamentals | Introductory+ | Can explain why it exists, when synchronous completion helps, why Task remains default, and the safe single-consumption rule |
| Thread Safety Fundamentals | Introductory+ | Can derive lost updates, identify the full read-modify-write critical section and explain basic `lock` ownership |

## Coding

| Pattern | Current Level | Evidence |
|---|---:|---|
| Hash Lookup | 8/10 | Solved Two Sum |
| Running Minimum | 8/10 | Solved Best Time to Buy/Sell Stock |
| Frequency Counting | 8/10 | Solved Valid Anagram |
| Prefix / Suffix | 7.5/10 | Solved Product Except Self with guidance and optimization |
| Two Pointers | 8.5/10 | Solved Valid Palindrome optimally |
| Fixed Sliding Window | 8/10 | Solved Maximum Average Subarray I; corrected indexing bug |
| Prefix Sum / Running Sum | 8/10 | Solved Find Pivot Index with O(n)/O(1) solution |
| Fast & Slow Pointers | 8.5/10 | Solved Linked List Cycle and Middle of the Linked List with O(n)/O(1) solutions |
| Linked List Pointer Manipulation | 8.7/10 | Solved Reverse Linked List and Merge Two Sorted Lists; learned dummy-head merge pattern |
| Stack / LIFO | 8.7/10 | Independently derived Valid Parentheses, Min Stack auxiliary state and RPN postfix evaluation |
| Trees / Graphs | Not started | User expects rustiness due to long gap |
| Dynamic Programming | Not started | Future topic |

## System Design

| Area | Current Level | Evidence |
|---|---:|---|
| Component Responsibility | 9/10 | Can explain Edge API/API Service/Orchestrator/TE/DE/Billing boundaries |
| Authentication Architecture | 9/10 | Can reason about registrations and security boundaries |
| Async Motivation | 8.5/10 | Can explain why long-running HTTP is a smell |
| Idempotency | 8.5/10 | Distinguishes unique constraints for creation, atomic stage claims for existing work, and optional Inbox for message-level dedupe |
| Queue Lifecycle | 8.7/10 | Can distinguish renewal, release-for-retry, completion, lock expiry and redelivery |
| Competing Consumers / Ordering | 8.3/10 | Understands queue ownership, duplicate stage messages, SQL claims and per-job dependency |
| Azure Service Bus Mapping | 8/10 | Mapped Peek-Lock, renew, complete, abandon/expiry and DLQ to the learned model |
| Failure Classification / DLQ | 8/10 | Distinguishes business validation from operational failures and controlled replay |
| Invoice Manager Async Topology | 8.2/10 | Selected separate OCR/field-extraction queues; Billing stays outside Orchestrator |
| Transactional Outbox | 8/10 | Can explain SQL update + queue publish gap, durable publish intent, duplicate publish and idempotent consumers |
| Notification Service design | Guided completion | Can explain producer/platform boundaries, tenant policy, per-channel fan-out, scheduling and observability; independent mock pending |
| Caching | Learned at building-block depth | Can choose cache/no-cache per data type, compare cache-aside/read-through/write strategies, and reason about TTL, L1/L2, eviction and invalidation |

## Communication

| Area | Current Level | Notes |
|---|---:|---|
| First-principles reasoning | Strong | User asks why technologies exist before choosing products |
| Trade-off articulation | Strong | Especially clear in SQL/Blob recovery, queue isolation and service ownership |
| Concise interview answers | Improving | Can now articulate durable success before acknowledgement and per-job ordering; continue refining into 30–60 second answers |
| Coding narration | Improving | Continue explaining invariants while coding |


## Session 10 Additions

| Skill | Status | Notes |
|---|---|---|
| Reverse linked list | Learned | Iterative O(1) solution and recursive interview version covered. |
| SQL update + queue publish gap | Learned | Can explain how a stage can get stuck when SQL commits but the next message is not published. |
| Transactional Outbox | Learned | Can explain durable publish intent, publisher lifecycle, duplicate publish, and idempotent consumer protection. |
| Full staged Invoice Manager pipeline | Learned | Upload/original PDF, PDF preparation, OCR, field extraction, user review/submit, billing boundary. |


## Session 11 Additions

| Area | Status | Evidence |
|---|---|---|
| Idempotent Consumer | Learned | Can explain duplicate delivery and how durable uniqueness prevents duplicate business work. |
| Unique constraint for initial creation | Learned | Can distinguish first-time insert races from existing-stage atomic updates. |
| Inbox pattern | Learned conceptually | Understands Inbox as optional generic message-level dedupe, distinct from business-table uniqueness. |
| Normalized job/stage schema | Learned decision | Chose `JobStages` to avoid widening `Jobs` for every stage and metadata field. |
| Merge Two Sorted Lists | Learned | Solved linked-list merge and learned dummy head / `dummy.next` return pattern. |


## Session 12 Additions

| Area | Status | Evidence |
|---|---|---|
| Backpressure | Learned | Can distinguish burst absorption from downstream capacity and branch by the actual bottleneck. |
| Per-pod bounded concurrency | Learned conceptually | Understands that an async gate can cap in-flight calls in one pod and that pod count multiplies total concurrency. |
| Global concurrency | Pending | Raised but intentionally deferred for dedicated coverage. |
| Remove Nth Node From End | Learned | Derived brute-force and one-pass two-pointer solutions; implemented with dummy head. |
| Production incident investigation | Practiced | Used Job ID/Correlation ID, stage state, queue signals and metrics → traces → logs. |
| Artifact/state reconciliation | Reinforced | Can identify Blob-written/SQL-stale failure window and avoid repeating OCR. |
| KEDA and Gunicorn | Mentioned, not covered | Only production context was discussed; internals remain pending. |


## Session 14 Additions

| Area | Status | Evidence |
|---|---|---|
| Notification Service | Guided completion | Produced an end-to-end workflow covering Invoice Manager event ownership, policy resolution, per-channel deliveries, reliability, scheduling and observability. |
| At-least-once vs resilience mechanisms | Clarified | Distinguished retry/redelivery/DLQ from outbox/inbox/idempotency/reconciliation. |
| Distributed scheduler | Learned conceptually | Identified duplicate selection, short atomic claims and transactional outbox creation. |
| Observability introduction | Learned conceptually | Proposed throughput, failure grouping and latency; expanded to queue depth, retry/DLQ and business identifiers. |
| Longest Repeating Character Replacement | Guided | Correct O(n)/O(1) code, but assistant revealed the optimal structure too early. |
| System-design interview process | Learning framework established | Requirement clarification → summary → assumptions → approach → responsibilities → architecture → deep dives → trade-offs → recap. |
| Detailed DR | Pending | Only high-level critical-state identification was discussed. |
| Circuit breaker | Not covered | Mentioned prematurely and explicitly parked. |


## Session 15 Additions

| Area | Status | Evidence |
|---|---|---|
| ValueTask fundamentals | Learned at introductory depth | Can explain synchronous-result optimization, hot-path criteria and profiling requirement. |
| Advanced ValueTask semantics | Pending | Multiple awaits, AsTask, IValueTaskSource and public API guidance intentionally deferred. |
| Stack / LIFO | Learned | Independently selected a stack for reverse-order bracket matching. |
| Valid Parentheses | Learned | Correct O(n)/O(n) implementation; review focused on idiomatic C# rather than algorithm repair. |
| Daily system-design cadence | Decided | One small building block per normal weekday session; Friday for complete simulations. |


## Session 16 Additions

| Skill | Status | Evidence |
|---|---|---|
| ValueTask safe consumption | Learned at usage level | Can state that a `ValueTask<T>` should be awaited once; internals remain pending. |
| Load Balancer fundamentals | Learned | Can explain motivation, routing strategies, health checks, L4/L7 selection and sticky-session trade-offs. |
| Stack with auxiliary state | Learned | Derived `(value, minAtThisLevel)` to make all Min Stack operations O(1). |


## Session 17 Additions

| Area | Status | Evidence |
|---|---|---|
| Thread safety fundamentals | Learned | Derived the lost-update race and identified the full read-modify-write sequence as the critical section. |
| Basic C# `lock` | Learned at introductory depth | Explained `private readonly` lock ownership and same-object mutual exclusion. |
| Evaluate Reverse Polish Notation | Learned | Correct O(n)/O(n) stack solution with correct operand ordering and integer division behavior. |
| Caching fundamentals | Learned | Covered cache-aside, read-through, write strategies, stale reads, stampede, L1/L2, TTL, LRU/LFU and warming. |
| Invoice Manager caching strategy | Learned | Chose selective caching for tenant/reference data and rejected caching large low-reuse OCR, result and PDF payloads. |
| Advanced synchronization/cache coherence | Pending | Explicitly outside current depth. |

## Session 18 Additions

| Area | Status | Evidence |
|---|---|---|
| Circuit breaker | Learned | Explained retry vs breaker, Closed/Open/Half-Open and failure classification. |
| Delayed queue retry | Learned conceptually | Selected delayed rescheduling for 40-second OCR work instead of immediate retry. |
| Recovery surge control | Learned conceptually | Identified overload/429 risk and proposed gradually increasing batches. |
| Adaptive rate/concurrency control | Pending | New concept was introduced but deliberately excluded from Session 18 coverage. |
| Monotonic stack | Learned at introductory depth | Derived unresolved-work stack and descending invariant from brute force. |
| Daily Temperatures | Learned | Correct index-based O(n) implementation and complexity explanation. |
