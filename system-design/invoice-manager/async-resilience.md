# Invoice Manager Async and Resilience Design Notes

## Scope

This file records the target-state concepts that have been derived so far. It does not describe features already present in production.

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

Target contract:

```http
HTTP/1.1 202 Accepted
```

```json
{
  "jobId": "job-123",
  "statusUrl": "/jobs/job-123/status"
}
```

The browser uses the job reference to query processing status. Polling is the simplest initial option; push-notification approaches remain a later design decision.

## Idempotency

The client sends one idempotency key per logical upload.

- Retry while the job is active → return the same `202`, `jobId` and status URL.
- Retry after completion → return the same logical outcome/result location.
- Same key with a different payload → reject as invalid reuse.

Correlation ID remains observability metadata and must not be reused as the idempotency key.

## Target State Responsibilities

```text
IM Web / Edge API
  → accepts user request and exposes status

IM REST API
  → validates request, creates job, produces initial work

Queue
  → durably hands work to a consumer

Orchestrator
  → consumes work, owns workflow-level queue interaction,
    updates workflow state and invokes TE/DE

Text Extractor
  → OCR and Azure AI abstraction

Data Extractor
  → field prediction
```

## Processing State and Artifacts

SQL is the authoritative store for future processing state. It should retain job lifecycle data and safe user-facing failure state.

Blob storage should hold temporary OCR artifacts. SQL stores the artifact reference and workflow metadata.

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

Do not mark a stage complete until its downstream work and the artifact required to advance/recover have both completed successfully.

## Failure Propagation

Store two distinct views of failures:

1. **Operational detail** for support/recovery: stage, attempts, category, technical error, correlation ID and timestamps.
2. **User-facing job status**: safe message, current stage and whether retry is meaningful.

Examples:

| Failure type | Retry | DLQ | User-facing result |
|---|---:|---:|---|
| Unsupported file | No | No | Upload a supported document. |
| Corrupted/unreadable document | No | No | The document could not be processed. |
| Temporary dependency failure | Yes | After retry limit | Processing is delayed or failed safely. |
| Azure AI 401/403 | No normal retry | Yes | Processing failed safely. |

## Current Open Boundary

The incremental design retains synchronous Orchestrator-to-TE/DE HTTP calls. Orchestrator is the queue consumer, so it owns temporary message ownership and its renewal while TE is processing.

Making TE or DE independent queue consumers is a larger architecture decision and remains open.

## Explicitly Deferred

- transactional consistency between SQL updates and next-stage work creation;
- product-specific queue configuration;
- notification transport choice;
- multi-region queued-work recovery;
- independent queue consumers for TE/DE.
