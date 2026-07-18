# Staff Interview Scorecard

## Session 17 Snapshot

| Area | Current assessment | Evidence |
|---|---:|---|
| Backend fundamentals | 9.2/10 | Derived race-condition mechanics and the complete read-modify-write critical section; advanced synchronization not yet assessed |
| Coding patterns | 9.0/10 | Identified and implemented RPN stack evaluation with correct operand order and no unnecessary defensive branches |
| System design | 9.3/10 | Made data-specific cache/no-cache decisions using reuse, size, freshness, security and durable-store trade-offs |
| Interview process | 9.0/10 | Clarified division and input validity, trusted the contract, and challenged caching trust-boundary implications |

## Session 17 Evidence

- Explained why protecting only the final write cannot prevent a lost update.
- Used one shared `private readonly` lock object to reason about mutual exclusion.
- Completed LeetCode 150 with O(n) time and O(n) space.
- Distinguished cache-aside from read-through based on application responsibility and attack surface.
- Derived L1/L2 flow, shorter L1 TTL, LRU/LFU trade-offs and cache warming.
- Rejected caching large, sensitive and low-reuse Invoice Manager payloads when Blob storage already satisfies durability and access.

## Session 16 Snapshot

| Area | Current assessment | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Retained `Task<T>` as default and added the safe single-consumption rule for `ValueTask<T>` without overclaiming deferred internals |
| Coding patterns | 8.9/10 | Identified the O(n) Min Stack bottleneck and derived per-level minimum state for O(1) operations |
| System design | 9.2/10 | Explained why load balancers exist and selected L4/L7 and routing strategies from required information |
| Interview process | 8.8/10 | Clarified constraints before Min Stack and challenged inconsistent assumptions during sticky-session discussion |

## Session 16 Evidence

- Distinguished a single server's capacity, availability and fairness problems.
- Derived round robin, least-load and geographic routing before terminology was introduced.
- Correctly selected Layer 7 for URL-based routing and Layer 4 for identical TCP backends.
- Derived the Min Stack auxiliary-state solution after identifying why one global minimum is insufficient.
- Required session-based rather than calendar-day-based feedback and progress references.

## Session 15 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Added introductory `ValueTask<T>` reasoning while keeping `Task<T>` as the default and advanced semantics pending |
| Coding patterns | 8.8/10 | Independently derived the optimal Stack/LIFO solution for Valid Parentheses |
| System design | 9.2/10 | No new building block completed; cadence was revised so small guided building blocks begin from Session 16 |
| Distributed systems | 8.9/10 | No new distributed-systems concept claimed in this session |
| Interview communication | 8.7/10 | Asked useful input and auxiliary-space questions before implementation |

## Session 15 Evidence

- Correctly identified the cache-hit path as the path that can benefit from `ValueTask<T>`.
- Correctly reasoned that `ValueTask<T>` has additional representation complexity and is not beneficial when work is predominantly asynchronous.
- Independently selected a stack for nested bracket validation and stated O(n) time and O(n) worst-case space.
- Implementation was algorithmically correct; review changes were idiomatic C# improvements (`TryGetValue`, `TryPop`, `Count == 0`).

This is a living review document. It tracks readiness by competency rather than topic count.

## Session 14 Snapshot

This snapshot records observed evidence only; it does not claim long-term improvement where the repository lacks a fair comparison.

| Competency | Status | Evidence |
|---|---|---|
| Guided system design | Completed for current scope | Explained producer/platform boundaries, per-channel fan-out, retries, scheduling, observability and Invoice Manager event ownership |
| Independent system-design interview | Not yet assessed | Full walkthrough was coached and the mock was stopped; Friday simulations will assess this later |
| Distributed-systems reasoning | Demonstrated | Correctly distinguished retry/DLQ for at-least-once attempts from outbox/inbox/idempotency/reconciliation for resilience |
| Coding implementation | Correct but guided | Longest Repeating Character Replacement compiled conceptually to O(n)/O(1), after the optimal structure was disclosed |
| Coding derivation process | Needs clean reassessment | Brute force and bottleneck derivation were skipped by the coach, so independent ability on this problem cannot be scored fairly |
| Requirement-gathering process | Being taught | Asked about burst load, SLA/SLO and FR/NFR; still in guided-learning phase rather than interview evaluation |
| Architecture boundary reasoning | Demonstrated | Corrected the event owner from Orchestrator to API Service based on ownership of final mapped/overridden extraction data |

## Session 14 Coaching Risks

- The coach gave away the coding solution too early.
- Circuit breaker and detailed DR concepts were introduced before coverage.
- Unsupported positive historical feedback was given and corrected.
- The design was declared complete before the user supplied the deep workflow.

These are process failures, not evidence of user weakness. `CURRICULUM.md` now governs future sessions.

## Session 13 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.5/10 | Can explain async I/O, state-machine transformation, `MoveNext()`, continuation scheduling, SynchronizationContext, ConfigureAwait and blocking risks |
| Coding patterns | 8.7/10 | Thirteen foundational problems; variable-size sliding window completed with HashSet and two forward-only pointers |
| System design | 9.2/10 | No new Session 13 system-design section; explicitly deferred rather than rushed |
| Distributed systems | 8.9/10 | Existing backpressure, outbox and idempotency foundation retained |
| Cloud architecture | 8.4/10 | Connected ASP.NET Core continuation behavior to ThreadPool scalability |
| Interview communication | 8.7/10 | Strong first-principles answers on ConfigureAwait and thread affinity; coding validation should be more systematic |

## Notable Improvements Since Session 12

- Built a detailed mental model of the compiler-generated async state machine.
- Distinguished Task, TaskScheduler, SynchronizationContext, ThreadPool and asynchronous I/O.
- Explained why ASP.NET Core has no request SynchronizationContext.
- Learned when independent I/O should be initiated before awaiting.
- Completed Longest Substring Without Repeating Characters using variable-size sliding window.
- Identified the need to validate algorithms with counterexamples before declaring complexity.

## Session 12 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Strong identity/backend foundation; dedicated .NET internals need to resume from Session 13 |
| Coding patterns | 8.6/10 | Twelve foundational problems; Remove Nth Node From End completed with one-pass reasoning |
| System design | 9.2/10 | Can investigate stage-wise, isolate bottlenecks and reason about backpressure |
| Distributed systems | 8.9/10 | Strong queue/outbox/idempotency/reconciliation foundation; global concurrency remains pending |
| Cloud architecture | 8.4/10 | Correctly separates internal saturation from external-provider rate limiting |
| Interview communication | 8.6/10 | Asked for Job ID/Correlation ID before investigating and explained branch decisions clearly |

## Notable Improvements Since Session 11

- Learned backpressure and bottleneck-oriented scaling.
- Completed Remove Nth Node From End and clarified that one pass does not mean one loop.
- Practiced metrics → traces → logs for production incident investigation.
- Reinforced reconciliation when durable Blob output exists but SQL state is stale.
- Explicitly separated production mentions of KEDA/Gunicorn from formal coverage.

## Session 11 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Strong HTTP, session, cookie, JWT, OAuth and OIDC understanding |
| Authentication and authorization | 9.5/10 | Can reason about scopes, OBO, service identity, Zero Trust and blast radius |
| Coding patterns | 8.5/10 | Eleven foundational problems; linked-list cycle, middle, reversal and merge covered |
| System design | 9.1/10 | Can derive durable workflow boundaries, stage ownership, outbox, idempotent consumers and normalized stage state |
| Distributed systems | 8.7/10 | Can reason about outbox, duplicate delivery, unique constraints, atomic claims, optional Inbox and at-least-once behavior |
| Cloud architecture | 8.4/10 | Mapped queue model to Service Bus and selected target queue topology |
| Interview communication | 8.5/10 | Good correction of schema assumptions and clearer distinction between creation vs existing-stage processing |

## Notable Improvements Since Session 10

- Learned that first-time job creation needs a database-enforced unique idempotency key.
- Clarified that atomic updates protect existing rows/stages but not insert races where no row exists yet.
- Distinguished business-table uniqueness from the Inbox pattern.
- Accepted normalized `Jobs` + `JobStages` target schema for Invoice Manager stage state.
- Solved Merge Two Sorted Lists and learned the dummy-head technique.
- Practiced mock interview explanation for duplicate queue messages.


## Session 10 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Strong HTTP, session, cookie, JWT, OAuth and OIDC understanding |
| Authentication and authorization | 9.5/10 | Can reason about scopes, OBO, service identity, Zero Trust and blast radius |
| Coding patterns | 8.4/10 | Ten foundational problems; linked-list cycle, middle and reversal covered |
| System design | 9.0/10 | Strong current-vs-target-state discipline; can derive durable workflow boundaries, stage ownership and outbox handoff |
| Distributed systems | 8.5/10 | Can reason about acknowledgement ambiguity, redelivery, competing consumers, atomic stage claims, outbox and idempotent consumers |
| Cloud architecture | 8.4/10 | Mapped first-principles queue model to Service Bus and selected target queue topology |
| Interview communication | 8.4/10 | Good self-correction and assumption stating; keep tightening wording around recoverable vs stuck states |

## Notable Improvements Since Session 9

- Solved Reverse Linked List iteratively with correct pointer preservation and O(1) extra space.
- Brushed up recursive reverse linked list, especially the base case and `head.next.next = head` unwinding step.
- Identified the SQL commit + queue publish failure window.
- Understood Transactional Outbox as durable publish intent stored in SQL.
- Understood outbox row lifecycle: Pending → Published → retained/archived/deleted later.
- Understood that outbox gives at-least-once publishing, not exactly-once publishing.
- Connected duplicate outbox publishes to idempotent consumers and atomic SQL stage claims.
- Corrected the Invoice Manager pipeline to include PDF preparation / page-image generation before OCR.
- Clarified that every stage completes only after durable output exists and SQL points to it.

## Current Weak Spots

- Global concurrency control, distributed rate limiting and detailed delayed retry/jitter.
- Deeper Inbox implementation details only if needed beyond the Session 11 concept.
- Multi-region queued-work recovery, including outbox behavior.
- Trees, graphs, heaps and DP.
- Networking module.
- Behavioral STAR stories.

## Coaching Notes

The user learns best when concepts are derived from concrete failure windows and then mapped to Invoice Manager. Follow the agreed curriculum in order; do not cut planned sections short or introduce future concepts as learned content.

## Session 18 Snapshot

| Competency | Current assessment | Evidence |
|---|---:|---|
| Backend fundamentals | 9.4/10 | Distinguished transient retry from persistent-failure circuit breaking and corrected breaker classification for `400`. |
| Coding patterns | 9.0/10 | Derived the monotonic-stack mechanism from O(n²) brute force and implemented Daily Temperatures in O(n). |
| Distributed systems | 9.0/10 | Chose durable delayed retry for OCR and identified thundering-herd/429 risk during recovery. |
| Interview communication | 8.9/10 | Explained index storage, value comparison and positional result clearly before coding. |

## Session 18 Evidence

- Corrected that `400` should not increment the circuit breaker.
- Proposed delayed rescheduling for a failed long-running OCR job.
- Identified that releasing 500 queued jobs could overwhelm recovery and suggested progressively larger batches.
- Derived stack choice, monotonic invariant and O(n) amortized reasoning before implementation.
- Produced correct C# using `TryPeek`, index storage and `i - index`.
