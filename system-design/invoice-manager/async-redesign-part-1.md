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

## Queue Placement

The queue must align with service responsibility rather than merely being placed at the earliest possible point.

### Queue after IM Web

This keeps the public request short, but IM Web should not own workflow-level progress.

### Queue after IM REST API — tentative direction

```text
IM Web / Edge API
  ↓
IM REST API
  ↓
Queue
  ↓
Orchestrator
  ↓
TE / DE
```

Why this direction is currently preferred:

- IM REST API is close to business validation and job creation.
- IM Web remains a thin public boundary.
- Orchestrator remains workflow owner.
- A queue decouples the browser request from long-running work.

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

Blob storage
- OCR JSON
- page-level OCR output
- extracted text
```

## Queue Lifecycle Learned So Far

- one consumer temporarily owns one received message;
- the message remains recoverable until processing is successfully completed;
- ownership must be renewed for long-running OCR;
- successful handling occurs only after the stage result required for recovery has been persisted;
- worker failures are classified before deciding between direct job failure, retry, or DLQ;
- operational replay must be deliberate and controlled.

## Open Questions

- Which Azure messaging product best fits the target design?
- Should one work item represent a whole job or a processing stage?
- How should product-specific queue settlement be configured?
- Should TE/DE later become independent queue consumers?
- How should the system make database state and next-stage work creation consistent?
