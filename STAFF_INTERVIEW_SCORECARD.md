# Staff Interview Scorecard

This is a living review document. It tracks readiness by competency rather than topic count.

## Day 8 Snapshot

| Competency | Current Level | Evidence |
|---|---:|---|
| Backend fundamentals | 9.3/10 | Strong HTTP, session, cookie, JWT, OAuth and OIDC understanding |
| Authentication and authorization | 9.5/10 | Can reason about scopes, OBO, service identity, Zero Trust and blast radius |
| Coding patterns | 8/10 | Eight foundational problems; structured brute-force-to-optimization flow is becoming consistent |
| System design | 8.6/10 | Strong ownership and current-vs-target-state discipline |
| Distributed systems | 7.6/10 | Derived queue lifecycle, retries, failure classification and DLQ replay policy |
| Cloud architecture | 8.2/10 | Strong Azure/AKS context; Azure messaging product mapping remains pending |
| Interview communication | 8.1/10 | Strong reasoning; keep answers concise and complete |

## Notable Improvements Since Day 7

- Derived queue primitives from failure modes rather than memorizing a cloud product.
- Identified that queue ownership belongs to the message consumer.
- Correctly challenged whether TE should be queue-aware in the incremental architecture.
- Improved failure taxonomy beyond a simplistic transient/permanent split.
- Added operational distinction: Azure AI 401/403 should be investigated and can go to DLQ even though retry is not useful.
- Added fast/slow pointer pattern with linked-list cycle detection.

## Current Weak Spots

- Azure Service Bus mapping and queue topology.
- Remaining queue concepts: topic/queue usage, ordering and competing consumers.
- Trees, graphs, heaps and DP.
- Networking module.
- Behavioral STAR stories.

## Coaching Notes

The user learns best when concepts are derived from problems and then mapped to Invoice Manager. Avoid vendor implementation details until the underlying abstraction is complete. Do not add future concepts to learned documentation before they are covered.
