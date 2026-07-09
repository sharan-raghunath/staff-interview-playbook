# Queue Fundamentals

## Status

Conceptual lifecycle and Azure Service Bus mapping complete for the current scope.

## Why a Queue Exists

A queue is a durable handoff mechanism between a producer and a consumer.

It is useful when the business operation outlives the transport request that started it.

For Invoice Manager:

```text
HTTP request creates a job
  ↓
HTTP returns 202 Accepted
  ↓
Durable work item waits for a background consumer
  ↓
Consumer processes the work independently of the browser
```

Core principle:

> The lifetime of an HTTP request must not dictate the lifetime of a long-running business process.

## Job State vs Queue Message

These solve different problems.

| Item | Purpose | Authoritative location |
|---|---|---|
| Job/stage state | User-visible processing lifecycle and recovery metadata | SQL |
| Queue message | Durable handoff of pending work to a consumer | Queue |
| OCR artifact | Recoverable intermediate processing output | Blob Storage, referenced by SQL |

Redis may accelerate frequently read status but must not be the authority for recovery-critical state.

## Producer and Consumer

```text
Producer
  creates durable work item
  ↓
Queue
  holds work until a consumer receives it
  ↓
Consumer
  owns and processes one work item
```

For the incremental Invoice Manager target state:

```text
IM REST API → produces work for a stage
Orchestrator → consumes OCR and field-extraction work
Text Extractor / Data Extractor → processing services called by Orchestrator
```

## Temporary Exclusive Ownership

A received message must not be deleted immediately. If the consumer crashes, the work must remain recoverable.

```text
Visible
  ↓
Received by consumer
  ↓
Hidden / owned temporarily
  ↓
Successful handling → remove from queue
Retryable failed attempt → release for later retry
Consumer failure or expired ownership → eligible for another consumer
```

## Three Consumer Actions

| Situation | Consumer action |
|---|---|
| Work is still progressing and needs more time | renew ownership |
| Current attempt failed and should retry later | release for retry |
| Durable stage result and SQL state prove success | complete / acknowledge |

Renewing ownership is not a business progress report. It says only that the consumer remains alive and owns the message.

For the incremental Invoice Manager target state, Orchestrator owns renewal because it consumes the queue message and calls TE/DE synchronously over HTTP.

## Durable Success Before Completion

For a processing stage, acknowledgement/completion comes after the recovery-critical output and SQL stage state are durable.

OCR example:

```text
Receive OCR message
→ call Text Extractor
→ persist OCR artifact in Blob Storage
→ update SQL OCR-stage state and artifact reference
→ complete queue message
```

Acknowledging early can lose work permanently. Acknowledging late can cause redelivery, which durable-state reconciliation can make safe.

## Redelivery and Reconciliation

A redelivery does not automatically mean repeat the work.

```text
1. Read SQL stage state.
2. If stage is complete, do not call the processor; complete the message.
3. If SQL is incomplete, inspect deterministic durable output for the same job/stage/idempotency identity.
4. If output exists, reconcile SQL and complete the message.
5. Only if neither SQL nor durable output proves completion, process the work.
```

This protects against crashes between artifact persistence, SQL updates, and queue completion. It also handles an ambiguous completion timeout: a completion request may have succeeded even though the worker did not receive the response.

## Competing Consumers and Duplicate Messages

Multiple Orchestrator pods can consume from the same queue. The queue gives one visible message to one consumer while its temporary ownership remains valid.

After lease expiry, another consumer can receive the message. A prior worker might later resume, so a lease alone is not enough for exactly-once business processing.

For duplicate messages of the same workflow stage, the consumer atomically claims the stage in SQL:

```text
Pending → Processing
only if it is still Pending.
```

The protections have separate roles:

```text
Queue lease
→ protects one received message from competing consumers.

Atomic SQL stage claim
→ protects one workflow stage from duplicate messages.

Idempotency key + durable-state checks
→ make retries and redelivery safe after crashes or ambiguous outcomes.
```

## Ordering

Invoice Manager requires ordering per job:

```text
OCR complete + durable OCR artifact
→ field extraction becomes eligible.
```

It does not require global FIFO across all invoices. Different jobs can be processed in parallel.

SQL stage transitions and atomic stage claims enforce the dependency. Strict ordered Service Bus sessions were discussed as a product capability but are not selected for this target design.

## Queue vs Topic

Use a queue for a work command that one logical worker must perform:

```text
Perform OCR for Job 123.
Extract fields for Job 123.
```

Use a topic when one published event must be independently delivered to multiple interested subscribers. A topic is not part of the OCR/field-extraction topology derived so far because these are work commands, not broadcast events.

## Azure Service Bus Mapping

| Concept | Azure Service Bus mapping |
|---|---|
| Durable work queue | Service Bus queue |
| Receive without immediate deletion | Peek-Lock receive mode |
| Temporary ownership | message lock |
| Extend ownership while work remains healthy | lock renewal (`RenewMessageLock`) |
| Durable success | `CompleteMessage` |
| Retryable failed attempt | `AbandonMessage` or lock expiry, followed by retry policy |
| Operationally unprocessable message | dead-lettering / DLQ |
| Multiple replicas share work | competing consumers |

Use explicit message settlement for the Invoice Manager stage workflow. Receive-and-delete semantics would remove a message before the stage's durable output and SQL state could prove success.

## Failure Handling

### Business validation failure

Examples:

- unsupported file format;
- corrupted document;
- invalid input payload.

Action:

```text
Fail job directly
Do not retry normally
Do not send to DLQ
Return safe user-facing guidance
```

### Retryable dependency failure

Examples:

- timeout;
- temporary network failure;
- temporary downstream unavailability;
- rate limiting / temporary overload.

Action:

```text
Record attempt as needed
Release for retry under backoff policy
Stop after maximum retry count
Move exhausted work to DLQ for investigation/replay
```

### Operational non-retryable failure

Examples:

- invalid credentials;
- wrong endpoint;
- missing permission;
- broken deployment configuration.

Action:

```text
Fail job safely
Capture technical detail
Dead-letter message
Alert and investigate
```

## Invoice Manager Queue Topology

```text
Durable PDF upload + SQL job state
  ↓
im-ocr-work
  ↓
Orchestrator → Text Extractor
  ↓
Blob OCR artifact + SQL OCR complete
  ↓
im-field-extraction-work
  ↓
Orchestrator → Data Extractor
  ↓
Durable prediction result + SQL ready-for-review state
```

Separate OCR and field-extraction queues isolate their distinct dependencies, scaling behavior, retry pressure, backlog signals, and DLQ operations.

## Explicitly Deferred

- deeper Inbox implementation details, if needed beyond the Day 11 concept;
- detailed delayed-retry implementation;
- backpressure implementation;
- Service Bus sessions as a selected design;
- multi-region queued-work recovery;
- Kafka comparison.

## SQL Update + Queue Publish Gap

A queue message may need to be created after a SQL state change.

Example:

```text
OCR artifact stored
→ SQL updated: OCR = Completed
→ field-extraction message should be published
```

If the service crashes after SQL commit but before queue publish, the next-stage trigger can be lost.

This is not solved by message acknowledgement alone because the missing message was never published.

## Transactional Outbox

The Transactional Outbox pattern records the intent to publish in SQL in the same transaction as the business state change.

```text
Same SQL transaction:
- update workflow state
- insert Outbox row for next message
```

A background publisher later reads pending outbox rows, publishes them to the queue, and marks them published.

This makes message publishing recoverable after service crashes.

Important trade-off:

> Outbox prevents lost messages, but publishing is at-least-once. Consumers must still be idempotent.

Duplicate messages are handled by checking SQL stage state and atomically claiming the stage before doing work.
