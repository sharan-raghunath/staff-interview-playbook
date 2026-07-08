# Invoice Manager Async and Resilience Design Notes

## Scope

This file records target-state concepts that have been derived so far. It does not describe features already present in production.

## Core Problem

Current Invoice Manager processing is synchronous and fail-fast. Long-running OCR and prediction are tied to the HTTP request that initiated them.

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
  → validates request, creates job, produces stage work

Service Bus queues
  → durable OCR and field-extraction work handoff

Orchestrator
  → consumes stage work, owns queue interaction,
    updates workflow state, invokes TE/DE

Text Extractor
  → OCR and Azure AI abstraction

Data Extractor
  → field prediction
```

## Processing State and Artifacts

SQL is the authoritative store for future processing state. It retains job lifecycle data and safe user-facing failure state.

Blob storage holds temporary OCR artifacts. SQL stores the artifact reference and workflow metadata.

Target workflow states may include:

```text
Accepted
Queued
OCRInProgress
OCRComplete
FieldInferenceInProgress
PredictionComplete
ReadyForUserReview
Submitted
Completed
Failed
```

## Settlement and Recovery

Orchestrator receives work under temporary ownership. It completes a message only after required durable output and SQL stage state are persisted.

For OCR:

```text
Receive message
→ TE returns output
→ persist Blob artifact
→ update SQL OCR state
→ complete message
```

A failure to confirm completion does not cause OCR to rerun. On redelivery, Orchestrator checks SQL first, then the deterministic OCR artifact location, and only calls TE when neither proves success.

Duplicate stage messages are protected by an atomic SQL claim such as `Pending → Processing` only when the stage is still pending.

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

```text
PDF/job durable
→ OCR queue
→ OCR durable completion
→ field-extraction queue
→ predictions ready
```

The ordering requirement is per job, not global FIFO across invoices. SQL stage state governs whether field extraction is eligible.

## Billing Boundary

After field extraction, the user reviews/overrides predictions. IM REST API owns the user-confirmed submission transition. Billing is initiated after that transition and remains outside Orchestrator's OCR/field-extraction responsibility.

## Explicitly Deferred

- transactional consistency between SQL updates and next-stage work creation;
- detailed delayed retry/backpressure implementation;
- Service Bus sessions as a selected design;
- multi-region queued-work recovery;
- independent queue consumers for TE/DE;
- billing queue implementation.
