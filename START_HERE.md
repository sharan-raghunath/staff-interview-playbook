# START HERE — Interview Prep Context

Read this file first when beginning a new chat in the Interview Prep project.

## Goal

Prepare for Senior / Staff Backend Engineer interviews with emphasis on:

- backend engineering;
- distributed systems;
- system design;
- cloud architecture;
- coding patterns;
- Staff-level communication.

Target companies include Atlassian, Microsoft, Adobe, SAP, Nutanix, Dassault Systèmes and Google-level opportunities.

## How to Work With This Repository

The repository is the durable source of truth. The chat is a working session.

Preferred session rhythm:

1. Backend / theory topic.
2. One coding pattern and one LeetCode problem.
3. System design using Invoice Manager.
4. One small system-design building block.
5. Interview-quality wording.
6. End-of-session repository update.

Coaching rules:

- Use first principles before vendor/product features.
- Derive concepts from the problem that created them.
- Keep current production behavior separate from proposed target state.
- Do not document a concept as learned until it has actually been covered.
- Follow the agreed plan in order; do not cut, skip, or deviate unless explicitly requested.
- Use the full hint ladder for coding; do not reveal the optimal invariant, formula or algorithm early.
- Teach the system-design interview process before judging interview performance.
- Feedback must cite concrete evidence.
- Do not introduce uncovered topics inside the current lesson.

## Completed Coding Problems

| Session | Problem | Pattern | Status |
|---:|---|---|---|
| 1 | Two Sum | Hash Lookup | ✅ |
| 2 | Best Time to Buy and Sell Stock | Running Minimum | ✅ |
| 3 | Valid Anagram | Frequency Counting | ✅ |
| 4 | Product of Array Except Self | Prefix / Suffix | ✅ |
| 5 | Valid Palindrome | Two Pointers | ✅ |
| 6 | Maximum Average Subarray I | Fixed-size Sliding Window | ✅ |
| 7 | Find Pivot Index | Prefix Sum / Running Sum | ✅ |
| 8 | Linked List Cycle | Fast & Slow Pointers | ✅ |
| 9 | Middle of the Linked List | Fast & Slow Pointers | ✅ |
| 10 | Reverse Linked List | Pointer Manipulation | ✅ |
| 11 | Merge Two Sorted Lists | Linked List Merge / Dummy Head | ✅ |
| 12 | Remove Nth Node From End | Two Pointers / Dummy Head | ✅ |
| 13 | Longest Substring Without Repeating Characters | Variable-size Sliding Window | ✅ |
| 14 | Longest Repeating Character Replacement | Variable-size Sliding Window | 🟡 Excessively guided |
| 15 | Valid Parentheses | Stack / LIFO | ✅ |
| 16 | Min Stack | Stack with Auxiliary State | ✅ |
| 17 | Evaluate Reverse Polish Notation | Stack / Postfix Evaluation | ✅ |
| 18 | Daily Temperatures | Monotonic Stack / Next Greater Event | ✅ |

## Completed Backend Topics

- HTTP statelessness;
- sessions and cookies;
- distributed sessions with Redis;
- JWT, JWKS, `kid`, claims and key rotation;
- roles/scopes and RBAC/ABAC;
- OAuth 2.0 and OIDC;
- Authorization Code, Implicit, Client Credentials and OBO concepts;
- ID token vs access token;
- refresh-token lifecycle, rotation/reuse-detection principles;
- logout, SSO/SLO;
- Zero Trust identity principles.
- async/await internals and ValueTask usage fundamentals.
- Thread-safety fundamentals: race conditions, critical sections and basic `lock`.
- Circuit breakers, retry classification, half-open recovery probes and controlled backlog ramp-up.

## Latest Completed Session

Session 18 completed:

- circuit breakers, retries, half-open probes and retry amplification;
- delayed queue retry for long-running OCR work;
- thundering-herd risk and controlled recovery;
- revision of idempotency, Inbox/Outbox, leases and reconciliation;
- LeetCode 739 — Daily Temperatures, derived through guided discovery.

## Canonical Invoice Manager Context

Invoice Manager predicts fields from an invoice, lets a user correct predictions, and submits the final reviewed result to the configured source system.

It is **not** the system of record for invoice business data.

Current production model:

```text
Client Web
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

- IM Web / Edge API is the public-facing service boundary and hosts Angular.
- Text Extractor performs document preprocessing and OCR.
- Text Extractor can use Azure Computer Vision or Azure Document Intelligence.
- Data Extractor performs ML/GenAI field prediction.
- Current processing is synchronous and fail-fast.
- Current persisted business-relevant data includes user prediction overrides used for pricing; it does not persist invoice business data as a system of record.

## Target-State Direction Learned So Far

```text
IM REST API stores original PDF and creates durable SQL job state
  ↓
Transactional Outbox: PdfPreparationRequested
  ↓
PDF preparation / page-image generation
  ↓
durable page/image artifacts + SQL references
  ↓
Transactional Outbox: OCRRequested
  ↓
Orchestrator → TE
  ↓
Blob OCR artifact + SQL OCR-complete state
  ↓
Transactional Outbox: FieldExtractionRequested
  ↓
Orchestrator → DE
  ↓
durable prediction output + ReadyForUserReview state
  ↓
User review / override / submit
  ↓
IM REST API initiates Billing independently, using final user-confirmed data
```

SQL is authoritative for target-state job/stage state. Redis remains a distributed cache, not a correctness boundary.

## Queue and Outbox Concepts Learned on Sessions 8–10

- HTTP lifetime and business-process lifetime should be independent.
- A long-running operation needs a Job ID and persistent SQL stage state.
- A queue is a durable handoff between producer and consumer.
- Received work needs temporary exclusive ownership rather than immediate deletion.
- Ownership can be renewed while a healthy long-running attempt continues.
- A consumer must distinguish renewal, release-for-retry, and completion.
- A stage is completed only after required durable result and SQL state are persisted.
- On redelivery, reconcile SQL and durable artifacts before repeating work.
- Multiple consumers safely share work through queue ownership plus atomic SQL stage claims.
- Invoice Manager needs per-job stage ordering, not global FIFO.
- OCR and field extraction use separate queues.
- Azure Service Bus mapping: Peek-Lock, lock renewal, Complete, Abandon/expiry, and DLQ.
- Business validation failures do not enter DLQ; operational failures and exhausted retry attempts can.
- DLQ replay follows root-cause remediation and runs in controlled batches.
- The SQL update + queue publish gap can lose the trigger for the next stage.
- Transactional Outbox records the next message-to-publish in the same SQL transaction as the stage-state update.
- Outbox gives at-least-once publishing; idempotent consumers protect against duplicate messages.
- Invoice Manager includes a PDF preparation / page-image generation stage before OCR.

## Session 11 Additions

- Idempotent consumer strategy is now covered.
- First-time job creation uses a database-enforced unique idempotency key, such as `UNIQUE(TenantId, IdempotencyKey)`.
- Existing stage processing uses an atomic `Pending → Processing` stage claim.
- Inbox pattern is understood as optional generic message-level deduplication when business tables are not sufficient.
- `JobStages` is now an accepted target-state schema decision for normalized stage state. This was introduced on Session 11; it was not assumed in earlier days.

## Session 12 Additions

- Backpressure and bottleneck-oriented scaling are covered.
- Per-pod bounded concurrency is understood conceptually; global concurrency remains deferred.
- LeetCode 19 was completed with brute-force and one-pass reasoning.
- The Friday mock reinforced Job ID/Correlation ID investigation, stage timelines, metrics → traces → logs, and durable-artifact reconciliation.
- TE/DE Gunicorn settings were discussed only as production context; KEDA and Gunicorn internals are not yet covered.

## Where to Continue Next

Session 18 is complete. Read `CURRICULUM.md`, `project/pending.md`, and `session-journal/session-018.md` before choosing Session 19 scope.
- Keep adaptive concurrency and distributed/adaptive rate limiting pending for a dedicated lesson.
- Continue the coding roadmap from the planned order; do not automatically repeat monotonic stack.
- Continue one small system-design building block in normal weekday sessions.
- Keep KEDA internals, Gunicorn internals and detailed DR in **mentioned but not covered** status until dedicated sessions.

Do not mark future topics as learned until covered.


## Session 14 Additions

- Guided design of a reusable multi-tenant Notification Service.
- Producer remains channel-agnostic; Notification Service owns policy and delivery.
- Invoice Manager API Service, not Orchestrator, emits the completion event because API owns final mapped/overridden business data.
- One independent delivery/outbox record per selected channel.
- Retry/DLQ separated from broader outbox/inbox/idempotency/reconciliation resilience.
- Scheduling uses UTC execution time with local/time-zone context retained when needed.
- Observability begins from business identifiers as well as technical correlation IDs.
- Detailed DR was pending at this point; circuit breaker was later completed in Session 18.
- Coding problem completed correctly but marked excessively guided.
- Friday is the preferred mock-interview day; weekends are optional.
- `CURRICULUM.md` is mandatory operating context.

Next session should begin only after reading the latest journal and pending list.


## Session 15 Additions

- `ValueTask<T>` fundamentals are covered; advanced usage restrictions remain pending.
- Valid Parentheses completed independently using a stack and closing-to-opening mapping.
- Stack/LIFO is now a learned coding pattern.
- Normal weekday sessions now include backend theory, coding, a small system-design building block and journal reconciliation.
- Friday remains interview-simulation and weekly-review day; weekends remain optional.


## Session 16 Additions

- `ValueTask<T>` should be treated as a single-consumption awaitable at the current curriculum depth.
- Load Balancer fundamentals completed: motivation, routing strategies, health checks, Layer 4 vs Layer 7 and sticky sessions.
- Min Stack completed with O(1) `Push`, `Pop`, `Top` and `GetMin` by storing the minimum at each stack level.
- Sessions are logical learning units and may span multiple calendar days; references and feedback must be session-based.

## Session 18 Additions

- Circuit breaker is now covered at interview depth.
- `400`, auth and business-validation failures are not retryable and do not increment the breaker.
- Long-running OCR work uses delayed queue retry rather than immediate worker retry.
- Recovery backlog must be ramped gradually to avoid a thundering herd.
- Adaptive rate limiting/concurrency remains pending as a dedicated topic.
- Daily Temperatures completed using an index-based monotonic stack.
- New coding patterns should be derived before they are named.

Next session: read `session-journal/session-018.md` and `project/pending.md`, then begin Session 19 on Monday.
