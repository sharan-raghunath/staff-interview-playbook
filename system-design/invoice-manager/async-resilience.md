# Invoice Manager Async and Resilience Design Notes

## Scope

This file records target-state concepts that have been derived so far. It does not describe features already present in production.

## Core Problem

Current Invoice Manager processing is synchronous and fail-fast. Long-running document preparation, OCR and prediction are tied to the HTTP request that initiated them.

Consequences:

- poor user experience while waiting;
- gateway/browser timeout risk;
- client disconnect may leave costly work with no response recipient;
- users may retry and duplicate work;
- no durable recovery path for partially completed processing.

Staff-level summary:

> A business operation can outlive its HTTP request, so the business operation should be decoupled from that request lifecycle.

## Async API Contract

```http
HTTP/1.1 202 Accepted
```

```json
{
  "jobId": "job-123",
  "statusUrl": "/jobs/job-123/status"
}
```

The browser polls status through IM Web/Edge. SQL is the authority for the status returned to the browser.

## Idempotency

The client sends one idempotency key per logical upload.

- Retry while the job is active → return the same `202`, `jobId`, and status URL.
- Retry after completion → return the same logical outcome/result location.
- Same key with a different payload → reject as invalid reuse.

Correlation ID remains observability metadata and must not be reused as the idempotency key.

## Target State Responsibilities

```text
IM Web / Edge API
  → accepts user request and exposes status

IM REST API
  → validates request, stores original PDF reference, creates job,
    and records first async work through the outbox

Service Bus queues
  → durable stage-work handoff

Outbox publisher
  → publishes pending outbox rows to queues

Orchestrator
  → consumes stage work, owns queue interaction,
    updates workflow state, invokes TE/DE

Text Extractor
  → OCR and Azure AI abstraction

Data Extractor
  → field prediction
```

## Full Staged Workflow

```text
Upload accepted
→ original PDF stored
→ PDF preparation / page-image generation
→ OCR
→ field extraction
→ user review / override
→ user submit
→ billing
```

The exact artifact format/path for PDF preparation and the exact field-extraction persistence model remain deferred.

## Durable Stage Completion Rule

A stage is completed only after its durable output exists and SQL points to it.

Common sequence:

```text
service returns result
→ durable output is stored
→ SQL transaction updates stage state and output references
→ outbox row is inserted for next async action, if any
→ current queue message is completed
```

Examples:

```text
PDF preparation completed
→ page images/artifacts stored + SQL references exist

OCR completed
→ OCR artifact stored + SQL OCR reference exists

Field extraction completed
→ prediction output stored + SQL ReadyForUserReview state exists
```

## Transactional Outbox

The outbox handles the reliability gap between SQL updates and publishing the next queue message.

Naive failure:

```text
SQL stage update commits
↓ crash
next queue message is never published
```

Outbox solution:

```text
Same SQL transaction:
- update stage state
- store durable output reference
- insert outbox row for the next message
```

A background publisher later sends the outbox message to Service Bus and marks the outbox row as published.

The outbox gives at-least-once publishing, so consumers must remain idempotent.

## Settlement and Recovery

Orchestrator receives work under temporary ownership. It completes a message only after required durable output and SQL stage state are persisted.

For OCR:

```text
Receive message
→ TE returns output
→ persist Blob artifact
→ SQL transaction:
   - OCR = Completed
   - artifact reference stored
   - FieldExtraction = Pending
   - Outbox row = FieldExtractionRequested
→ complete OCR message
```

A failure to confirm completion does not cause OCR to rerun. On redelivery, Orchestrator checks SQL first, then the deterministic OCR artifact location, and only calls TE when neither proves success.

Duplicate stage messages are protected by an atomic SQL claim such as `Pending → Processing` only when the stage is still pending.

Session 11 clarified the duplicate-safety split:

```text
Initial job creation
→ Jobs table unique constraint: UNIQUE(TenantId, IdempotencyKey)

Existing stage processing
→ atomic claim on normalized JobStages state

Generic message-level deduplication
→ optional Inbox table when business tables are not enough
```

Target schema direction: normalize per-stage status and metadata into `JobStages` rather than adding one set of columns per stage to `Jobs`.

## Failure Propagation

Store two distinct views of failures:

1. **Operational detail** for support/recovery: stage, attempts, category, technical error, correlation ID, and timestamps.
2. **User-facing job status**: safe message, current stage, and whether retry is meaningful.

Examples:

| Failure type | Retry | DLQ | User-facing result |
|---|---:|---:|---|
| Unsupported file | No | No | Upload a supported document. |
| Corrupted/unreadable document | No | No | The document could not be processed. |
| Temporary dependency failure | Yes | After retry limit | Processing is delayed or failed safely. |
| Azure AI 401/403 | No normal retry | Yes | Processing failed safely. |

## Queue Placement and Ordering

Accepted queues for current scope:

```text
im-ocr-work
im-field-extraction-work
```

A PDF-preparation work item exists conceptually before OCR, but exact queue naming/topology remains deferred.

The ordering requirement is per job, not global FIFO across invoices. SQL stage state governs whether the next stage is eligible.

## Billing Boundary

After field extraction, the user reviews/overrides predictions. IM REST API owns the user-confirmed submission transition. Billing is initiated after that transition and remains outside Orchestrator's OCR/field-extraction responsibility.

If billing is asynchronous, the user-submission transaction should insert `BillingRequested` into the outbox.

## Explicitly Deferred

- deeper Inbox implementation details, if needed beyond the Session 11 concept;
- detailed delayed retry/backpressure implementation;
- Service Bus sessions as a selected design;
- multi-region queued-work recovery;
- independent queue consumers for TE/DE;
- exact PDF-preparation artifact format/storage path;
- detailed field-extraction output persistence;
- billing queue implementation.
