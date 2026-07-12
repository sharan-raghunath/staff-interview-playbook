# ADR-014 — Normalize Job Stage State

## Status

Accepted as target-state direction on Session 11.

## Context

Invoice Manager's target async workflow has multiple stages:

```text
Upload
→ PDF preparation
→ OCR
→ field extraction
→ user review / submit
→ billing
```

Earlier sessions established that SQL must be authoritative for job and stage state, but the physical schema had not been finalized.

One option is to store every stage field directly on `Jobs`:

```text
PdfPreparationStatus
OcrStatus
FieldExtractionStatus
BillingStatus
...
```

As the workflow grows, each stage would require more columns for status, attempts, owner, timestamps, errors and artifact references.

## Decision

Normalize stage state into a separate `JobStages` table for the target design.

Conceptual model:

```text
Jobs
- JobId
- TenantId
- IdempotencyKey
- OriginalPdfReference
- OverallStatus
- CreatedAt
- UpdatedAt

JobStages
- JobStageId
- JobId
- StageName
- Status
- ArtifactReference
- AttemptCount
- ProcessingOwner
- StartedAt
- CompletedAt
- LastErrorCode
- LastErrorMessage
```

Current uniqueness:

```text
Jobs: UNIQUE(TenantId, IdempotencyKey)
JobStages: UNIQUE(JobId, StageName)
```

## Consequences

### Positive

- Adding a new stage does not require widening the `Jobs` table.
- Stage retry metadata has a natural home.
- Atomic stage claims become generic across stages.
- Operational queries become easier, such as all failed OCR stages or stale Processing stages.

### Negative

- More schema complexity.
- More joins when reading full job status.
- The system must coordinate overall job status with per-stage status.

## Important Note

`JobStages` was introduced and accepted as a Session 11 schema decision. It was not an assumed design in Sessions 8–10.
