# Invoice Manager Async Redesign — Part 1

## Current State

Invoice Manager currently performs prediction synchronously.

```text
IM Web / Edge API
  ↓
IM REST API
  ↓
Orchestrator
  ↓
Text Extraction
  ↓
Data Extraction
  ↓
Response to user
```

Characteristics:

- Fail-fast.
- No queue.
- No retry mechanism.
- No persisted workflow checkpoints.
- Long-running processing is coupled to the HTTP request lifetime.
- If a downstream service fails or the orchestrator crashes, the user sees a failure and must retry manually.

## Target-State Motivation

Large documents can take minutes to process. A long-running business operation should not depend on a browser request or gateway timeout.

Target API behavior:

```text
POST /predict
  ↓
Validate request
  ↓
Create processing job
  ↓
Enqueue command
  ↓
Return 202 Accepted + jobId + statusUrl
```

## Tentative Queue Placement

The queue should not be placed merely where it is easiest technically. It should align with service responsibilities.

Options discussed:

### Queue after IM Web

Pros:

- Keeps user-facing request short.
- IM Web need not understand downstream failures.

Cons:

- IM Web should not own detailed workflow progress.
- IM Web cannot meaningfully know whether OCR is complete or field inference is pending.

### Queue after IM REST API

Pros:

- IM REST API is closer to business validation and job creation.
- IM REST API can persist initial job state.
- Orchestrator remains the workflow owner.
- User-facing HTTP flow is still decoupled from long-running processing.

Tentative target direction:

```text
IM Web / Edge API
  ↓
IM REST API
  ↓
Queue
  ↓
Orchestrator
  ↓
Text Extraction / Data Extraction
```

Final technology and topology are intentionally pending until queue fundamentals are covered.

## Job State Store

SQL should be the authoritative store for job state.

Why SQL:

- Durable.
- Consistent.
- Suitable for audit and troubleshooting.
- Does not lose state due to cache eviction or TTL.
- Better for recovery-critical workflow data.

Redis may be used for frequently polled status or transient cache, but not as the source of truth.

Principle:

> SQL is the source of truth for processing state. Redis is an optimization layer.

## Candidate Job States

```text
Accepted
Queued
PickedUpForOCR
OCRInProgress
OCRComplete
FieldInferenceInProgress
PredictionComplete
ReadyForUserReview
SubmittedToSourceSystem
Completed
Failed
DeadLettered
```

Stage-completion rule:

> Mark a stage complete only after the downstream operation has successfully completed and the required artifacts have been persisted.

## OCR Artifact Persistence

In the target async design, OCR output should be persisted as a processing artifact.

Reason:

If OCR completes but the orchestrator crashes before field inference, the system should resume from OCR complete rather than call Text Extraction again.

Suggested split:

```text
SQL
- JobId
- Current stage
- OCR artifact URI
- checksum/hash
- timestamps
- failure details

Blob Storage
- OCR JSON
- page-level OCR output
- extracted text
```

This does not make Invoice Manager the system of record for invoices. It only stores temporary processing artifacts needed for recovery.

## Pending

Before finalizing the queue design, the next session should derive queue fundamentals:

- Producer
- Consumer
- Message ownership
- ACK / NACK
- Message lock / visibility timeout
- Retry
- DLQ

Then map those concepts to Azure Service Bus, Kafka and RabbitMQ.
