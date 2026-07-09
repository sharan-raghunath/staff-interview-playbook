# Staff Interview Scorecard

This is a living review document. It tracks readiness by competency rather than topic count.

## Day 11 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Strong HTTP, session, cookie, JWT, OAuth and OIDC understanding |
| Authentication and authorization | 9.5/10 | Can reason about scopes, OBO, service identity, Zero Trust and blast radius |
| Coding patterns | 8.5/10 | Eleven foundational problems; linked-list cycle, middle, reversal and merge covered |
| System design | 9.1/10 | Can derive durable workflow boundaries, stage ownership, outbox, idempotent consumers and normalized stage state |
| Distributed systems | 8.7/10 | Can reason about outbox, duplicate delivery, unique constraints, atomic claims, optional Inbox and at-least-once behavior |
| Cloud architecture | 8.4/10 | Mapped queue model to Service Bus and selected target queue topology |
| Interview communication | 8.5/10 | Good correction of schema assumptions and clearer distinction between creation vs existing-stage processing |

## Notable Improvements Since Day 10

- Learned that first-time job creation needs a database-enforced unique idempotency key.
- Clarified that atomic updates protect existing rows/stages but not insert races where no row exists yet.
- Distinguished business-table uniqueness from the Inbox pattern.
- Accepted normalized `Jobs` + `JobStages` target schema for Invoice Manager stage state.
- Solved Merge Two Sorted Lists and learned the dummy-head technique.
- Practiced mock interview explanation for duplicate queue messages.


## Day 10 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Strong HTTP, session, cookie, JWT, OAuth and OIDC understanding |
| Authentication and authorization | 9.5/10 | Can reason about scopes, OBO, service identity, Zero Trust and blast radius |
| Coding patterns | 8.4/10 | Ten foundational problems; linked-list cycle, middle and reversal covered |
| System design | 9.0/10 | Strong current-vs-target-state discipline; can derive durable workflow boundaries, stage ownership and outbox handoff |
| Distributed systems | 8.5/10 | Can reason about acknowledgement ambiguity, redelivery, competing consumers, atomic stage claims, outbox and idempotent consumers |
| Cloud architecture | 8.4/10 | Mapped first-principles queue model to Service Bus and selected target queue topology |
| Interview communication | 8.4/10 | Good self-correction and assumption stating; keep tightening wording around recoverable vs stuck states |

## Notable Improvements Since Day 9

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

- Backpressure/concurrency control and detailed delayed retry.
- Deeper Inbox implementation details only if needed beyond the Day 11 concept.
- Multi-region queued-work recovery, including outbox behavior.
- Trees, graphs, heaps and DP.
- Networking module.
- Behavioral STAR stories.

## Coaching Notes

The user learns best when concepts are derived from concrete failure windows and then mapped to Invoice Manager. Follow the agreed curriculum in order; do not cut planned sections short or introduce future concepts as learned content.
