# Invoice Manager Future State

## Goal

Move long-running invoice processing away from a purely synchronous request-response model.

The user should not wait for the entire OCR + extraction workflow to complete within a single HTTP request.

---

## Proposed Direction

Introduce an asynchronous boundary around the orchestration layer.

Possible high-level flow:

```text
User
↓
Edge API
↓
API Service / Invoice Processing Service
↓
Persist job
↓
Queue
↓
Worker / Orchestrator
↓
Text Extractor
↓
Data Extractor
↓
Persist result
↓
Notify user / allow polling
```

---

## Why Async?

Current synchronous design has limitations:

- long-running requests
- gateway timeouts
- poor user experience for large documents
- limited recovery if Orchestrator crashes
- retries may duplicate expensive work
- progress is not clearly persisted

Async design can improve:

- resilience
- retry handling
- user experience
- throughput
- recovery
- scalability

---

## User Experience

Instead of waiting for final extraction result:

1. User uploads document.
2. System returns `202 Accepted`.
3. User receives a tracking/job ID.
4. UI shows processing status.
5. User is notified when processing completes.

Notification options:

- polling
- WebSockets
- SignalR
- Server-Sent Events

TODO:

- Learn WebSockets and SignalR internals before deciding final approach.

---

## Required Capabilities

Future async architecture needs:

- durable job state
- idempotency keys
- stage-level checkpoints
- retry policy
- dead letter queue
- status tracking
- result persistence
- observability
- timeout handling
- recovery after crash

---

## Orchestrator Failure Scenario

Example:

```text
Text Extractor completes
↓
Orchestrator crashes before Data Extractor starts
```

Future behaviour should be:

- OCR completion state is persisted
- Orchestrator/worker can resume from the next stage
- Data Extractor is invoked once or idempotently
- duplicate retries do not corrupt state
- user sees job status rather than 500 error

---

## Open Questions

- Where should workflow state be stored?
- Should the queue message represent the whole invoice or one processing stage?
- Should TE and DE publish events directly?
- Should Orchestrator remain central or become a workflow worker?
- What is the retry policy per stage?
- What qualifies for DLQ?
- How do we handle partial success?
- How do we notify users?
- How do we handle multi-region DR with queued jobs?
