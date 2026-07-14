# Interview Roadmap

## Phase 0 — Staff Engineer Thinking

- [x] Responsibilities over technologies
- [x] Trade-off based reasoning
- [x] First-principles learning
- [x] Production-system anchoring
- [x] Security-boundary based reasoning
- [ ] STAR story extraction
- [ ] Staff-level communication drills

## Phase 1 — Backend Engineering Mastery

- [x] APIs
- [x] HTTP Statelessness
- [x] Sessions
- [x] Cookies
- [x] Distributed Sessions
- [x] JWT
- [x] OAuth 2.0
- [x] OIDC
- [x] Token Lifecycle
- [x] Logout / SSO / SLO
- [ ] SAML
- [ ] CORS
- [ ] CSRF
- [ ] DNS
- [ ] TLS
- [ ] HTTP/1.1
- [ ] HTTP/2
- [ ] HTTP/3
- [ ] REST vs gRPC
- [ ] Reverse Proxies
- [ ] Load Balancers
- [ ] L4 vs L7
- [ ] ASP.NET Middleware
- [ ] Dependency Injection
- [ ] Garbage Collection
- [ ] Threading
- [x] async/await internals
- [x] Task<T> vs ValueTask<T> fundamentals
- [ ] Advanced ValueTask usage semantics

## Phase 2 — Coding Patterns

- [x] Hash Lookup
- [x] Running Minimum
- [x] Frequency Counting
- [x] Prefix / Suffix
- [x] Two Pointers
- [x] Fixed-size Sliding Window
- [x] Prefix Sum / Running Sum
- [x] Fast & Slow Pointers
- [x] Variable-size Sliding Window
- [x] Stack
- [ ] Queue
- [ ] Binary Search
- [x] Linked List: reversal and manipulation
- [x] Linked List: merge with dummy head
- [ ] Trees
- [ ] BST
- [ ] Heap
- [ ] Graphs
- [ ] Backtracking
- [ ] Dynamic Programming

## Phase 3 — Distributed Systems

- [x] Idempotency introduction
- [x] Retry strategy introduction
- [x] Queue fundamentals from first principles
- [x] Visibility timeout and ownership renewal concepts
- [x] Failure classification
- [x] DLQ and controlled replay policy
- [x] Exponential backoff concept
- [x] Queue completion, release-for-retry, and redelivery reconciliation
- [x] Azure Service Bus mapping for current scope
- [x] Queue vs topic
- [x] Competing consumers and per-job ordering
- [x] Atomic stage claim as an idempotent-consumer protection
- [x] Backpressure and per-pod bounded concurrency
- [ ] Global concurrency control
- [ ] Detailed delayed-retry mechanics
- [x] Transactional messaging problem: SQL update + queue publish gap
- [x] Outbox pattern
- [x] Unique constraint for first-time idempotent creation
- [x] Normalized job/stage state schema decision
- [x] Idempotent consumer / Inbox pattern concept
- [ ] Multi-region queued-work recovery
- [ ] CAP theorem
- [ ] Consistency models
- [ ] Eventual consistency
- [ ] At-least-once vs exactly-once processing as a formal guarantee model
- [ ] Kafka
- [ ] Saga pattern
- [ ] CQRS
- [ ] Event sourcing
- [ ] Distributed locking
- [ ] Leader election
- [ ] Service mesh
- [x] Observability introduction through Notification Service
- [ ] Observability dedicated backend topic

## Phase 4 — Cloud Architecture

- [ ] AKS
- [ ] KEDA
- [ ] HPA
- [ ] App Gateway
- [ ] Azure Front Door
- [ ] API Management
- [ ] Azure Functions
- [ ] App Service
- [ ] Azure SQL
- [ ] Cosmos DB
- [ ] Redis
- [ ] Key Vault
- [ ] Azure Monitor
- [ ] Networking
- [ ] AWS comparison
- [ ] GCP comparison

## Phase 5 — System Design

- [x] URL Shortener introduction
- [x] Invoice Manager architecture baseline
- [x] Invoice Manager authentication architecture
- [x] Invoice Manager async motivation
- [x] Async API contract
- [x] Idempotency / retry / DLQ introduction
- [x] Invoice Manager async redesign Part 1
- [x] Queue lifecycle for Invoice Manager target state
- [x] Service Bus mapping, queue choice, and stage topology
- [x] Billing boundary after user-confirmed submission
- [x] Invoice Manager full staged async pipeline through billing boundary
- [ ] Invoice Manager end-to-end async redesign with backpressure and DR
- [x] Notification Service guided design
- [x] Reusable system-design interview process
- [ ] Friday system-design simulation
- [ ] E-commerce
- [ ] Chat system
- [ ] File upload system
- [ ] Payment gateway
- [ ] Banking system
- [ ] Video streaming
- [ ] Google Docs
- [ ] Uber
- [ ] Dropbox
- [ ] Stock trading

## Phase 6 — Behavioral

- [ ] Biggest failure
- [ ] Biggest success
- [ ] Conflict resolution
- [ ] Leading without authority
- [ ] Handling outages
- [ ] Architecture decisions
- [ ] Mentoring
- [ ] Trade-offs
- [ ] Delivering under pressure

## Phase 7 — Company-specific Prep

- [ ] Atlassian
- [ ] Microsoft
- [ ] Adobe
- [ ] SAP
- [ ] Nutanix
- [ ] Dassault Systèmes
- [ ] Google


## Session 12 — Completed

- Backpressure and bottleneck-oriented scaling.
- Per-pod bounded concurrency concept.
- Production-context discussion of TE sync and DE `gthread` worker choices.
- LeetCode 19: Remove Nth Node From End.
- Friday production-incident mock.

## Session 13 — Completed / Partially Deferred

- Completed `async`/`await` internals, state machine, `MoveNext()`, SynchronizationContext, ConfigureAwait, blocking deadlocks and TaskScheduler.
- Completed LeetCode 3: Longest Substring Without Repeating Characters.
- System design was explicitly deferred and then continued as guided Notification Service work in Session 14. Mock interview remained deferred.


## Session 14 — Completed

- Guided multi-tenant Notification Service design.
- Clarified Invoice Manager API Service as owner of the final business result and completion event.
- Fan-out into one delivery/outbox record per channel.
- Retry, DLQ, inbox/outbox, idempotency and reconciliation distinctions.
- Scheduling and distributed scheduler claim/outbox flow.
- Observability introduction through logs and metrics.
- High-level regional-failure boundary only; detailed DR remains pending.
- Longest Repeating Character Replacement completed with excessive guidance.
- Added reusable system-design interview process and coaching contract.
- Friday selected as preferred interview-simulation day; weekends optional.


## Session 15 — Completed

- `Task<T>` versus `ValueTask<T>` fundamentals.
- Profiling-driven criteria for considering `ValueTask<T>`.
- Valid Parentheses completed using Stack/LIFO.
- Added durable notes for `ValueTask`, stack fundamentals and Valid Parentheses.
- Revised Monday–Thursday cadence to include one small system-design building block per session.
- Friday retained as interview-simulation and weekly-review day.
