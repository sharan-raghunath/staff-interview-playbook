# Session 14 — Notification Service and Guided Sliding Window

## Session context

Session 14 was an optional Sunday session and was intentionally spread across multiple sittings. The user clarified that weekends are optional and that the curriculum should extend over a longer period rather than overload each day alongside a full-time job.

The repository and journal terminology changed from **Day** to **Session**. A session remains open until its planned items are completed or explicitly deferred.

## Planned work

- Guided system design: multi-tenant Notification Service.
- Coding: Longest Repeating Character Replacement.
- No new backend-theory topic.
- No mock interview; full simulations are now preferably reserved for Friday.

## Coding — Longest Repeating Character Replacement

The problem was understood correctly using examples such as `ABA, k = 1` and `ABCA, k = 2`.

The final solution used:

- variable-size sliding window;
- a 26-entry uppercase frequency array;
- maximum character frequency;
- shrinking while required replacements exceed `k`.

The implementation was correct and O(n)/O(1).

### Coaching correction

The assistant revealed the optimal formula, variables and shrinking logic before asking for brute force and allowing the user to derive the optimization. The user explicitly corrected this.

The problem is therefore recorded as **completed with excessive guidance**. Future coding must follow:

1. understand and clarify;
2. brute force;
3. complexity;
4. repeated-work/bottleneck analysis;
5. hint ladder;
6. optimized implementation;
7. correctness and edge-case review.

## Guided system design — Notification Service

### Requirements and scope

The design supports:

- Email, SMS and Push;
- future WhatsApp extension;
- asynchronous processing;
- multi-tenancy;
- user preferences and opt-out;
- templates;
- retries;
- scheduled notifications;
- operational status and audit support.

Azure Service Bus was identified as the likely Azure messaging option for reliable queues/topics, but the design was kept responsibility-first.

### Producer and platform boundaries

Invoice Manager sends one channel-neutral business event. The Notification Service owns channel selection, tenant policy, user preferences, recipients, templates, providers and delivery.

The initial design discussion temporarily mixed Data Extractor behavior with Notification Service behavior. This was corrected.

A key Invoice Manager ownership correction was made:

- Orchestrator coordinates Text Extractor and Data Extractor.
- Invoice Manager API Service applies final mappings/overrides to predictions and owns the final business representation.
- Therefore the API Service should emit `InvoiceProcessingCompleted` only after the final result is persisted and ready for the user.
- The Notification Service should receive only the data needed to render the notification, not the full extracted JSON or an internal storage dependency.

### Fan-out

One logical notification request is accepted, then the Notification Orchestrator resolves tenant policy and user preferences and creates independent delivery jobs.

One delivery/outbox record per channel was selected because each channel becomes independently retryable, observable and auditable.

### Reliability

The session distinguished two groups clearly.

At-least-once processing attempts rely on:

- durable broker redelivery;
- retry policy;
- delayed retry where appropriate;
- DLQ/permanent-failure handling after exhaustion.

A resilient end-to-end workflow additionally uses:

- transactional outbox;
- inbox/deduplication backed by stable IDs and unique constraints;
- idempotency;
- reconciliation.

The DLQ preserves failed work but does not guarantee successful business delivery.

### Failure handling and provider failover

- Temporary timeout: retry with delay.
- `429`: delayed retry.
- Invalid request: permanent/non-retryable failure record.
- Ambiguous provider response: stable provider idempotency key when supported, otherwise uncertain state and reconciliation.
- Provider failover depends on business criticality: critical notifications can move quickly to the secondary provider; non-critical traffic can retry the primary first and switch after a configured threshold.

Circuit breaker was introduced by the assistant without prior coverage. The user stopped the discussion, and the concept remains pending rather than learned.

### Scheduling

Scheduled delivery uses UTC for execution and indexing. Original local time and time-zone ID are retained when useful, especially for recurring schedules and audit.

Multiple scheduler instances can read the same due rows. The guided solution was to atomically claim a short batch, commit quickly and create an outbox record transactionally so a scheduler crash does not silently lose the work.

The outbox publisher can still republish after an ambiguous acknowledgement, which is why downstream inbox/idempotency remains required.

### Observability

Support should be able to trace from business identifiers such as tenant ID, invoice ID, customer/user ID and event type, not only an internal Notification ID.

Logs should carry non-PII correlation and delivery fields. Metrics discussed:

- total notifications;
- failures grouped by error, channel, provider, tenant and event type;
- average, P95 and P99 latency;
- queue depth;
- retry rate;
- DLQ depth;
- oldest-message age;
- overdue scheduled notifications.

### Regional-failure discussion boundary

Only high-level recovery considerations were discussed. The user correctly noted that the critical data/state must survive, and questioned whether delivered-notification history is pipeline-critical.

The answer depends on audit, support, compliance and billing requirements. Caches are normally reconstructable rather than correctness-critical.

Detailed DR, including RPO/RTO and active-active/active-passive design, was not formally covered and remains pending.

## System-design coaching process correction

The user has not previously answered system-design interviews and requested coaching on how to start and structure the thought process.

A reusable flow was agreed:

1. clarify requirements;
2. summarize the design-shaping requirements;
3. state material assumptions;
4. explain the approach;
5. identify responsibilities;
6. build high-level architecture;
7. deep-dive known concepts;
8. discuss trade-offs;
9. summarize end to end.

A full interview simulation was started prematurely and then stopped. Interview simulations will preferably happen on Friday after guided learning.

## Coaching failures explicitly recorded

During Session 14, the assistant:

- gave away the coding solution too early;
- introduced circuit breaker before it was covered;
- expanded into detailed DR concepts before the planned DR topic;
- gave unsupported historical feedback suggesting the user previously relied on technology names;
- marked the design walkthrough complete before the user had provided the deep architecture;
- relied too much on conversational momentum rather than the repository curriculum.

The user explicitly called these out. `CURRICULUM.md` was added as the operating contract to prevent repetition.

## Session 14 status

Completed:

- guided Notification Service design to the agreed current depth;
- Longest Repeating Character Replacement implementation, marked as excessively guided;
- reusable system-design interview process;
- migration from Day to Session terminology;
- coaching contract and repository reconciliation.

Deferred/pending:

- independent revisit of the coding pattern with a fresh problem;
- Friday interview simulation;
- dedicated DR topic;
- circuit breaker;
- detailed template lifecycle/versioning.
