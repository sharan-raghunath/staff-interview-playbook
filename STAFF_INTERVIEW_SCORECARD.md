# Staff Interview Scorecard

This is a living review document. It tracks readiness by competency rather than topic count.

## Day 9 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Strong HTTP, session, cookie, JWT, OAuth and OIDC understanding |
| Authentication and authorization | 9.5/10 | Can reason about scopes, OBO, service identity, Zero Trust and blast radius |
| Coding patterns | 8.2/10 | Nine foundational problems; fast/slow pointers used for cycle detection and midpoint finding |
| System design | 8.9/10 | Strong current-vs-target-state discipline; can derive durable workflow boundaries and stage ownership |
| Distributed systems | 8.2/10 | Can reason about acknowledgement ambiguity, redelivery, competing consumers, atomic stage claims, and per-job ordering |
| Cloud architecture | 8.4/10 | Mapped first-principles queue model to Service Bus and selected target queue topology |
| Interview communication | 8.3/10 | Increasingly concise and accurate; keep stating assumptions and ownership boundaries |

## Notable Improvements Since Day 8

- Distinguished lease renewal, release-for-retry, and successful message completion.
- Explained why completion must follow durable output and SQL stage state.
- Correctly handled ambiguous acknowledgement outcomes without rerunning expensive work.
- Derived competing consumers and identified that queue ownership alone is insufficient for duplicate stage messages.
- Added atomic SQL stage claims alongside idempotency and durable-state reconciliation.
- Distinguished per-job sequencing from unnecessary global FIFO.
- Mapped the learned model to Azure Service Bus terminology.
- Finalized separate OCR and field-extraction queues and preserved Billing outside Orchestrator.
- Added a second fast/slow-pointer application: middle of linked list.

## Current Weak Spots

- Backpressure/concurrency control and detailed delayed retry.
- Transactional messaging/outbox.
- Multi-region queued-work recovery.
- Trees, graphs, heaps and DP.
- Networking module.
- Behavioral STAR stories.

## Coaching Notes

The user learns best when concepts are derived from problems and then mapped to Invoice Manager. Follow the agreed curriculum in order; do not cut planned sections short or introduce future concepts as learned content.
