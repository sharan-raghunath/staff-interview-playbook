# Project Context

This file captures the durable working context for the Interview Prep project.

## Goal

Prepare for Senior / Staff Backend Engineer interviews at companies such as Atlassian, Microsoft, Adobe, SAP, Nutanix, Dassault Systèmes and Google-level opportunities.

## Coaching Style

Use first-principles reasoning rather than isolated definitions.

For every concept, derive:

1. What problem existed?
2. Why did the previous solution fall short?
3. What abstraction solves it?
4. What trade-offs does it introduce?
5. How should it be explained in a Staff interview?

Modes:

- **Learn Mode** — derive concepts with low interview pressure.
- **Interview Mode** — concise answer plus follow-up and wording refinement.
- **Architect Mode** — responsibilities, trade-offs and operations.

Hard rules:

- Separate current production behavior from target state.
- Do not add uncovered concepts to documentation except as titles in roadmap/pending lists.
- Follow the agreed curriculum in order. Do not cut planned sections short, skip items, or deviate unless the user explicitly asks.
- The repository reflects the user’s learned understanding, not generic best-practice leakage.

## Session Structure

- Monday–Thursday: one backend topic, one coding problem, one small system-design building block, and repository/journal reconciliation.
- Friday: coding interview, system-design interview, behavioral practice and weekly review.
- Saturday–Sunday: optional.
- A session closes only when planned work is complete or explicitly deferred.
- Read `CURRICULUM.md` before coaching.

## Coding Coaching

Always ask for brute force and complexity before optimization. Use a Hint Ladder:

- Hint 0: no hints;
- Hint 1: clarification;
- Hint 2: identify bottleneck/repeated work;
- Hint 3: point to pattern family;
- Hint 4: explain the algorithm only when needed.

## Completed Coding Problems

| Session | Problem | Pattern | Notes |
|---:|---|---|---|
| 1 | Two Sum | Hash Lookup | Complement lookup, dictionary, O(n) |
| 2 | Best Time to Buy and Sell Stock | Running Minimum | Track minimum price so far |
| 3 | Valid Anagram | Frequency Counting | Count character occurrences |
| 4 | Product of Array Except Self | Prefix / Suffix | Reuse output for O(1) extra space |
| 5 | Valid Palindrome | Two Pointers | O(n), skip non-alphanumeric |
| 6 | Maximum Average Subarray I | Fixed-size Sliding Window | Maintain window sum and max sum |
| 7 | Find Pivot Index | Prefix Sum / Running Sum | Compare left/right using total sum |
| 8 | Linked List Cycle | Fast & Slow Pointers | Floyd’s cycle detection, O(1) extra space |
| 9 | Middle of the Linked List | Fast & Slow Pointers | Find second middle in one traversal |
| 10 | Reverse Linked List | Pointer Manipulation | Iterative and recursive reversal |
| 11 | Merge Two Sorted Lists | Linked List Merge / Dummy Head | Reused nodes, dummy head, O(n+m)/O(1) |
| 12 | Remove Nth Node From End | Two Pointers / Dummy Head | Brute force and one-pass gap; one pass does not mean one loop |
| 13 | Longest Substring Without Repeating Characters | Variable-size Sliding Window | HashSet and two forward-only pointers |
| 14 | Longest Repeating Character Replacement | Replacement-budget Sliding Window | Correct but excessively guided |
| 15 | Valid Parentheses | Stack / LIFO | Independently derived optimal algorithm |
| 16 | Min Stack | Stack with Auxiliary State | Derived O(1) aggregate-per-level design |

## Backend Progress

Completed:

- APIs and ASP.NET Core request pipeline;
- HTTP statelessness;
- sessions, cookies and distributed sessions / Redis;
- JWT, claims, JWKS, `kid`, key rotation, roles/scopes and RBAC/ABAC;
- refresh-token lifecycle, rotation/reuse-detection principles, and revocation trade-offs;
- OAuth 2.0, OIDC, Authorization Code, Implicit, Client Credentials and OBO concepts;
- logout, SSO/SLO and identity best practices;
- Zero Trust identity principles;
- `async`/`await` internals, SynchronizationContext, ConfigureAwait and TaskScheduler;
- `Task<T>` versus `ValueTask<T>` fundamentals.

Pending:

- SAML;
- TLS, DNS, HTTP/2, HTTP/3, gRPC;
- CORS, CSRF, mTLS and proof-of-possession topics.

## Invoice Manager Context

Invoice Manager is the primary production example.

### Functional Purpose

Invoice Manager:

- accepts invoice documents;
- performs OCR;
- predicts fields through ML and/or GenAI;
- lets the user review and override predictions;
- submits final reviewed results to a configured source system.

It is **not** the system of record for invoice business data. The source system owns invoice lifecycle and business persistence.

### Current State

- Processing is synchronous and fail-fast.
- No queue or durable async job workflow exists today.
- The platform currently persists user overrides used for pricing plus operational/application metadata.
- A failure after expensive downstream work can still yield a failed request to the user.

### Target-State Direction Learned So Far

- The long-running business process should be decoupled from the browser HTTP request.
- API returns `202 Accepted`, `jobId`, and `statusUrl`.
- SQL is the authority for processing job/stage state.
- Redis is a distributed cache and not the correctness boundary.
- Service Bus queues provide durable stage-work handoff to Orchestrator.
- Orchestrator is the incremental target-state consumer and workflow owner for OCR and field extraction.
- TE/DE stay HTTP services in the initial migration; they do not need queue awareness.
- OCR artifacts should be persisted temporarily for recovery efficiency.
- Separate OCR and field-extraction queues isolate distinct dependencies and operations.
- Queue completion follows durable output and SQL state; redelivery reconciles before repeating work.
- Billing follows user-confirmed submission and remains outside Orchestrator.
- Target schema now normalizes workflow stage state into `JobStages`; this was introduced and decided on Session 11.
- Failure propagation separates operational detail from user-facing status.

### Canonical Architecture

```text
Client Web / Asset Finance Web
  ↓
Akamai
  ↓
Azure Application Gateway
  ↓
Traefik
  ↓
IM Web / Edge API
  ↓
IM REST API
  ↓
Orchestrator
  ├── Text Extractor
  └── Data Extractor
```

Important production details:

- Client identity can federate through Client IdP → FIS IdP → Microsoft Entra.
- The platform is multi-tenant and tenants can target different source systems.
- IM Web/Edge API, IM REST API, Orchestrator, Billing, TE and DE are application-owned services.
- Azure SQL, Redis, Key Vault, App Configuration, App Insights, Log Analytics, Azure CV and Azure DI are platform dependencies.
- Text Extractor abstracts Azure Computer Vision and Azure Document Intelligence.

## Queue / Messaging Concepts Learned

- producer/consumer durable handoff;
- temporary exclusive ownership and renewal for healthy long-running work;
- complete vs release-for-retry vs ownership renewal;
- safe acknowledgement after durable result and SQL stage state;
- redelivery reconciliation and ambiguous acknowledgement outcomes;
- competing consumers and lease expiry;
- atomic SQL stage claim for duplicate messages;
- per-job stage ordering without global FIFO;
- queue vs topic distinction;
- Azure Service Bus Peek-Lock, lock renewal, complete, abandon/expiry, and DLQ mapping;
- separate OCR and field-extraction queue topology;
- business validation vs retryable dependency vs operational failure;
- DLQ and controlled replay;
- SQL update + queue publish gap;
- Transactional Outbox for durable publish intent;
- at-least-once publishing and idempotent consumer protection;
- full staged Invoice Manager pipeline including PDF preparation before OCR;
- unique constraint for first-time job creation idempotency;
- Inbox pattern as optional generic message-level deduplication;
- normalized `Jobs` + `JobStages` target schema decision.

## Session 12 Progress

- Backpressure and bottleneck-oriented scaling are covered.
- Per-pod bounded concurrency is understood conceptually through an async concurrency gate; strict global concurrency remains deferred.
- TE sync and DE `gthread` configuration were discussed as production evidence only. Gunicorn itself is not yet formally covered.
- KEDA was mentioned as a possible scaling mechanism but not studied.
- LeetCode 19 was completed.
- Friday mock covered stage-wise investigation, Job ID/Correlation ID, metrics → traces → logs, and reconciliation when Blob output exists but SQL is stale.

## Immediate Next Work

- Session 16 is complete. Session 17 should follow the normal cadence: backend topic, coding problem, small system-design building block and journal update.
- Continue advanced `ValueTask<T>` only within the explicit pending scope.
- Continue the stack roadmap with a fresh problem.
- Begin the first dedicated small system-design building block.
- Keep global concurrency, distributed rate limiting, KEDA internals, Gunicorn internals, circuit breaker and detailed DR pending until dedicated coverage.
- Inbox pattern is covered conceptually; revisit only for deeper implementation detail if needed.
- Derive detailed field-extraction output persistence and Billing execution topology later.

## User Strengths

- Strong production architecture intuition.
- Thinks in responsibilities and trade-offs.
- Strong .NET/Azure background.
- Comfortably challenges assumptions and future leakage.
- Learns effectively through Socratic derivation.
- Asks Staff-level questions around identity, operations and failure handling.

## Areas to Improve

- Concise interview communication.
- Coding breadth across trees, graphs, DP, heaps and harder variants.
- Continue STAR-story preparation.

## Repository Workflow

1. User uploads the current repository ZIP.
2. Assistant reviews and updates files in place.
3. Assistant removes archive/OS/git artifacts from the deliverable.
4. Assistant verifies the documentation does not introduce uncovered concepts as learned content.
5. Assistant returns a replacement ZIP for the user to push to GitHub.

## Session 13 Progress

- Completed .NET `async`/`await` internals from first principles.
- Covered Task versus Thread versus Process, incomplete versus already-completed awaits, compiler-generated state machine, state field, awaiter, `MoveNext()` and `AsyncTaskMethodBuilder`.
- Covered independent I/O initiation, `Task.WhenAll`, SynchronizationContext, ASP.NET Core's lack of request context, ConfigureAwait, `.Result`/`.Wait()` deadlocks and TaskScheduler.
- Completed LeetCode 3 using a HashSet-backed variable-size sliding window.
- Reinforced algorithm validation against counterexamples before declaring complexity.
- System design and mock interview were explicitly deferred by the user and remain incomplete.


## Session 14 Progress

- Guided design of a multi-tenant Notification Service.
- Producer sends a channel-neutral event; Notification Service resolves tenant policy, recipients, preferences, templates and channels.
- One independent delivery/outbox record is created per selected channel.
- At-least-once attempts were separated from the wider resilience set of outbox, inbox, idempotency and reconciliation.
- Scheduling uses UTC execution time and atomic claiming plus outbox publication.
- Observability uses business identifiers as well as technical correlation IDs.
- Invoice Manager API Service owns final prediction mapping/overrides and emits the completion event after final business data is persisted. Orchestrator remains responsible for coordinating TE/DE.
- Detailed DR and circuit breaker remain pending and must not be used as learned concepts.
- Longest Repeating Character Replacement was implemented correctly but after the assistant revealed the optimal structure; record it as excessively guided.
- Friday is the preferred interview-simulation day; weekends are optional.
- `CURRICULUM.md` is now the authoritative coaching contract.


## Session 15 Progress

- Covered `Task<T>` versus `ValueTask<T>` fundamentals from first principles.
- Default remains `Task<T>`; `ValueTask<T>` is considered only for profiled hot paths with frequent synchronous completion.
- Advanced `ValueTask<T>` semantics remain pending.
- Completed Valid Parentheses independently using a stack and a closing-to-opening dictionary.
- Added stack/LIFO as a coding pattern.
- Revised normal weekday cadence to include one small system-design building block every session; Friday remains simulation/application day.


## Session 16 Progress

- Added the safe single-consumption rule for `ValueTask<T>` without introducing deferred implementation internals.
- Completed Load Balancer fundamentals: motivation, routing strategies, health checks, Layer 4 versus Layer 7 and sticky sessions.
- Completed Min Stack by storing `(value, minimumAtThisLevel)` for O(1) operations.
- Added the project rule that sessions may span multiple calendar days and all progress references must remain session-based.


## Session 17 Context

- Backend: race conditions, read-modify-write critical sections, mutual exclusion and basic `lock` with a `private readonly` object. Advanced synchronization remains pending.
- Coding: LeetCode 150 Evaluate Reverse Polish Notation completed in O(n) time/O(n) space with correct operand order and valid-input contract discipline.
- System design: caching fundamentals completed, including cache-aside, read-through, write-through/write-behind concepts, invalidation, stampede, per-key coordination, L1/L2, TTL, LRU/LFU, warming and distributed invalidation at a conceptual level.
- Invoice Manager decisions: cache tenant configuration and small reference data selectively; do not cache OCR payloads, final extraction outputs by default, or uploaded PDFs. Durable Blob storage remains the source for large workflow artifacts.
