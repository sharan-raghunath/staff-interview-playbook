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

- fail-fast;
- no queue;
- no durable workflow state;
- no retry/recovery path for processing stages;
- long-running work is coupled to the HTTP request lifetime.

## Target-State Motivation

Large documents can take minutes. The user-facing HTTP request should return quickly while processing continues independently.

```text
POST /predict
  ↓
Validate request
  ↓
Create job
  ↓
Create work item
  ↓
Return 202 Accepted + jobId + statusUrl
```

## Queue Placement and Stage Topology

The target direction is now:

```text
IM Web / Edge API
  ↓
IM REST API
  ↓
Create durable job / uploaded-document reference
  ↓
im-ocr-work
  ↓
Orchestrator → TE
  ↓
Blob OCR artifact + SQL OCR complete
  ↓
im-field-extraction-work
  ↓
Orchestrator → DE
  ↓
SQL prediction-ready state
```

The queues are placed after IM REST API because the internal API owns request validation and job creation, while Orchestrator owns asynchronous workflow execution.

Separate OCR and field-extraction queues give stage-level isolation for scaling, dependencies, retry pressure, backlog metrics, and DLQ handling.

## Job State Store

SQL is the authoritative store for target-state job status. Redis can accelerate read-heavy status access but must not be the source of truth.

## OCR Artifacts

Persist OCR output as a temporary processing artifact so a later-stage failure does not require re-running OCR unnecessarily.

Suggested split:

```text
SQL
- JobId
- current status/stage
- OCR artifact URI
- timestamps
- failure metadata
- stage claim/attempt metadata

Blob storage
- OCR JSON
- page-level OCR output
- extracted text
```

## Queue Lifecycle

- one consumer temporarily owns one received message;
- the message remains recoverable until processing is successfully completed;
- ownership is renewed for healthy long-running work;
- a retryable failed attempt is released for retry rather than treated as success;
- successful handling occurs only after the stage result required for recovery and SQL state are persisted;
- redelivery reconciles durable state before repeating expensive processing;
- atomic SQL stage claims prevent duplicate stage messages from starting the same stage concurrently.

## User Review and Billing

Field extraction prepares predictions for user review. User overrides are persisted and used for pricing.

```text
Predictions ready
→ user review / overrides
→ IM REST API persists submitted state
→ Billing initiated independently
```

Orchestrator does not own Billing.

## Open Questions

- How should the system make database state and next-stage work creation consistent?
- What detailed delayed-retry and backpressure policy should apply?
- Should TE/DE later become independent queue consumers?
- What durable representation should field-extraction results use?
- What is the Billing execution topology?
