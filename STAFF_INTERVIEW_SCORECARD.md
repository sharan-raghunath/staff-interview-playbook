# Staff Interview Scorecard

This is a living review document. It tracks readiness by competency rather than topic count.

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
- Inbox pattern.
- Multi-region queued-work recovery, including outbox behavior.
- Trees, graphs, heaps and DP.
- Networking module.
- Behavioral STAR stories.

## Coaching Notes

The user learns best when concepts are derived from concrete failure windows and then mapped to Invoice Manager. Follow the agreed curriculum in order; do not cut planned sections short or introduce future concepts as learned content.
