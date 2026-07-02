# Invoice Manager Future State

## Goal

Move long-running invoice processing away from a purely synchronous request-response model.

The user should not wait for the entire OCR + extraction workflow to complete within a single HTTP request.

---

## Proposed Direction

Introduce an asynchronous boundary around the processing workflow.

Possible high-level flow:

```text
User
↓
Edge API / Web App
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
Persist prediction result
↓
User reviews and overrides
↓
Submit to source system
```

---

## Why Async?

Current synchronous design has limitations:

- long-running requests
- gateway timeouts
- poor user experience for large documents
- limited recovery if Orchestrator crashes
- retries may duplicate expensive work
- progress is not durably tracked

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
5. User is notified or can fetch results when processing completes.

Notification options:

- polling
- long polling
- Server-Sent Events
- WebSockets
- SignalR

---

## Required Capabilities

Future async architecture needs:

- durable job state
- idempotency keys
- stage-level checkpoints
- retry policy
- dead letter queue
- status tracking
- temporary result persistence
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

## Open Design Questions for Day 7

- Azure Service Bus or Kafka?
- Queue or topic?
- Should queue messages represent whole jobs or individual stages?
- Where should workflow state be stored?
- Should TE/DE publish events or only respond to Orchestrator?
- Should Orchestrator be a stateful workflow engine or a stateless worker with external state?
- How should DLQ replay work?
- How should polling or push notification be selected?
- How should multi-region DR handle in-flight messages?
