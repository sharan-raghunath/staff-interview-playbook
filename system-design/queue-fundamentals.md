# Queue Fundamentals

## Status

Conceptual lifecycle complete. Azure Service Bus mapping is intentionally pending.

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
| Job state | User-visible processing lifecycle and recovery metadata | SQL |
| Queue message | Durable handoff of pending work to a consumer | Queue |
| OCR artifact | Recoverable intermediate processing output | Blob storage, referenced by SQL |

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
IM REST API → produces initial processing work
Orchestrator → consumes it and owns the workflow-level queue interaction
Text Extractor / Data Extractor → remain processing services called by Orchestrator
```

## Exclusive Temporary Ownership

A received message must not be deleted immediately. If the consumer crashes, the work must remain recoverable.

The queue therefore temporarily hides the message from other consumers while one consumer owns it.

```text
Visible
  ↓
Received by consumer
  ↓
Hidden / owned temporarily
  ↓
Successful handling → remove from queue
Consumer failure or expired ownership → eligible for another consumer
```

## Visibility Timeout and Ownership Renewal

The temporary ownership period is the visibility timeout.

If OCR takes longer than that period, the consumer must renew ownership before it expires. Otherwise another consumer may receive the same message while the original consumer is still working.

Lease renewal means:

> I am still alive and still own this work item.

It is not a business-progress update.

In the incremental Invoice Manager target state, Orchestrator owns renewal because it is the queue consumer. TE does not need queue awareness while Orchestrator invokes TE synchronously over HTTP.

## Successful Handling and Acknowledgement

A consumer needs a way to tell the queue:

> This work item was successfully handled; do not deliver it again.

Conceptually, that is an **acknowledgement**. The product-specific operation name is deliberately deferred.

For an OCR stage, the consumer should acknowledge/complete the queue work only after:

1. OCR succeeds.
2. The OCR artifact/reference required for recovery is persisted.
3. The job state reflects the completed stage.
4. The next stage work has been created.

The atomicity of persisting workflow state and creating the next work item is intentionally deferred to the transactional-messaging module.

## Failure Handling

The worker must classify failure before deciding what happens next.

### User input / business validation failures

Examples:

- unsupported file format;
- corrupted document;
- invalid input payload.

Action:

```text
Fail job directly
Do not retry
Do not send to DLQ
Return safe user-facing guidance
```

### Retryable dependency failures

Examples:

- timeout;
- temporary network failure;
- temporary downstream unavailability;
- throttling.

Action:

```text
Retry after delay
Use maximum retry count
Increase delay between repeated retries
```

### Operational failures requiring investigation

Examples:

- Azure AI authentication/authorization failure;
- configuration error;
- unexpected defect;
- retry budget exhausted.

Action:

```text
Fail job safely for user
Preserve technical context
Place message in DLQ
Alert and investigate
```

## Exponential Backoff

Repeated retry delays should increase rather than remain fixed.

```text
Attempt 1 → short wait
Attempt 2 → longer wait
Attempt 3 → longer again
```

Reason: a struggling dependency needs time to recover; immediate repeated retries can make the outage worse.

## Dead Letter Queue and Replay

DLQ is for messages requiring operational investigation or manual recovery. It is not a bucket for ordinary invalid user input.

Messages in DLQ should retain operational context such as:

```text
JobId
MessageId
Stage
AttemptCount
FailureCategory
Technical error detail
CorrelationId
Timestamp
```

Replay policy:

1. Diagnose the root cause.
2. Fix it.
3. Replay intentionally in batches.
4. Monitor outcomes and stop when the batch fails again.

## User-Facing Failure Propagation

The status API exposes safe, actionable data only.

```json
{
  "jobId": "job-123",
  "status": "Failed",
  "stage": "TextExtraction",
  "message": "We could not process this document.",
  "canRetry": false
}
```

Technical details belong in operational records and DLQ metadata, not browser responses.

## Deferred Topics

These are intentionally not explained here yet:

- Azure Service Bus product mapping and SDK APIs;
- product-specific completion/failure settlement names;
- queue vs topic;
- ordering and sessions;
- Kafka/RabbitMQ comparison;
- transactional messaging;
- outbox/inbox;
- saga.
