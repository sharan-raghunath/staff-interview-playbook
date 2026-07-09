# Invoice Manager Future State

## Goal

Move long-running invoice processing away from a purely synchronous request-response model while preserving a clear separation between Invoice Manager processing state and source-system invoice ownership.

This file describes the target-state direction learned so far. It is not current production behavior.

## Target Workflow Learned So Far

The workflow is staged. It does not jump directly from upload to OCR; there is also a PDF preparation / page-image generation stage before OCR.

```text
User
  ↓
IM Web / Edge API
  ↓
IM REST API
  ↓
Store original PDF durably + create SQL job state
  ↓
Transactional Outbox: PdfPreparationRequested
  ↓
PDF preparation / page-image generation
  ↓
Durable page/image artifacts + SQL references
  ↓
Transactional Outbox: OCRRequested
  ↓
Orchestrator → Text Extractor
  ↓
Blob OCR artifact + SQL OCR-complete state
  ↓
Transactional Outbox: FieldExtractionRequested
  ↓
Orchestrator → Data Extractor
  ↓
Durable prediction output + ReadyForUserReview state
  ↓
User reviews predictions / overrides values
  ↓
IM REST API persists final submission
  ↓
Billing initiated independently of Orchestrator
```

## API Experience

The request returns quickly with `202 Accepted`, a `jobId`, and a `statusUrl`.

The browser polls the job-status endpoint through IM Web/Edge. IM Web reads authoritative processing state from SQL. Polling is the initial correctness mechanism; push notification is not needed for correctness and remains a later design choice.

## Persisted State

SQL should be authoritative for:

- job identity;
- current stage and status;
- idempotency metadata;
- artifact references;
- safe user-facing failure status;
- operational failure metadata required for recovery;
- atomic stage claim state where duplicate work messages are possible;
- outbox rows representing messages that must be published.

## Target SQL Shape Learned on Day 11

Earlier sessions established authoritative stage state in SQL. Day 11 added the physical schema decision to normalize stage state rather than widening the `Jobs` table for every stage.

Conceptual `Jobs` table:

```text
Jobs
- JobId
- TenantId
- IdempotencyKey
- OriginalPdfReference
- OverallStatus
- CreatedAt
- UpdatedAt
- SubmittedAt
```

Important uniqueness:

```text
UNIQUE(TenantId, IdempotencyKey)
```

Conceptual `JobStages` table:

```text
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
- CreatedAt
- UpdatedAt
```

Current uniqueness:

```text
UNIQUE(JobId, StageName)
```

`JobStages` was introduced and accepted on Day 11. It was not assumed in earlier days.

Redis can cache status reads but cannot be the recovery authority.

## Queue and Consumer Boundary

The accepted target work queues so far are:

```text
im-ocr-work
im-field-extraction-work
```

A PDF-preparation work item also exists conceptually, but exact queue naming/topology for that stage has not yet been finalized.

Orchestrator consumes document-processing queues and invokes TE/DE synchronously over HTTP in the incremental design. It owns message lock renewal and settlement because it is the queue consumer. TE/DE remain business-processing services rather than queue-aware workers.

Separate OCR and field-extraction queues were selected because OCR and field extraction have distinct dependencies, scaling behavior, retry pressure, and operational visibility needs.

## Stage Success and Recovery

For each stage:

```text
Receive message under temporary ownership
→ atomically claim stage where applicable
→ process work
→ persist required durable output
→ SQL transaction:
   - update stage state and output references
   - create next-stage outbox row, if another async action is required
→ complete queue message
```

A stage is not complete just because a service returns a response. It is complete only when the stage's durable output exists and SQL points to it.

Examples:

```text
PDF preparation complete
→ page images/artifacts stored + SQL references exist

OCR complete
→ OCR artifact stored + SQL OCR reference exists

Field extraction complete
→ prediction output stored + SQL prediction-ready state exists
```

A redelivery first reconciles SQL state and durable output. It repeats processing only when neither proves the stage completed.

## Transactional Outbox

The outbox solves the failure gap between a SQL update and publishing the next queue message.

Example failure without outbox:

```text
OCR artifact stored
→ SQL says OCR completed
→ crash before FieldExtractionRequested is published
```

The OCR output is not lost, but the trigger for field extraction is missing.

With outbox:

```text
Same SQL transaction:
- OCR = Completed
- store OCR artifact reference
- FieldExtraction = Pending
- Outbox row = FieldExtractionRequested
```

A background outbox publisher sends pending outbox messages to Service Bus and marks them as published.

The outbox gives at-least-once publishing. If the publisher sends a message but crashes before marking the row published, the message may be sent again. Consumers remain safe through SQL stage checks, atomic stage claims, idempotency keys and, where needed, an Inbox table.

Key line:

> Outbox prevents lost messages. Idempotent consumers protect against duplicate messages.

## Ordering

The required order is per job:

```text
PDF preparation complete
→ OCR eligible
→ OCR completed and artifact durable
→ field extraction eligible
→ predictions ready for review
```

No global FIFO order across invoices is required. SQL stage state and atomic stage claims enforce workflow progression.

## Billing Boundary

Billing is not an Orchestrator stage.

Pricing depends on final user-reviewed values and overrides, so it is initiated only after the user submits through IM REST API.

When billing is asynchronous, the submit transaction should use the same outbox rule:

```text
Same SQL transaction:
- persist final submitted invoice/override state
- Billing = Pending
- Outbox row = BillingRequested
```

Billing queue implementation details remain deferred.

## Failure Policy

- User input errors fail the job directly; they do not enter the DLQ.
- Retryable dependency errors release/retry with increasing delay and a maximum retry count.
- Operational/system failures requiring investigation enter the DLQ.
- DLQ replay happens only after root-cause investigation and in controlled batches.

## Azure Service Bus Mapping

The current target mapping is Service Bus queues using explicit Peek-Lock settlement:

```text
message lock → temporary ownership
RenewMessageLock → extend ownership while healthy work continues
CompleteMessage → remove only after durable stage success
Abandon / lock expiry → retryable attempt returns for retry
Dead-letter → operational investigation path
```

## Deferred Decisions

- deeper Inbox implementation details, if needed beyond the Day 11 concept;
- detailed delayed-retry/backpressure design;
- Service Bus sessions as a selected ordering mechanism;
- exact PDF-preparation artifact format/storage path;
- detailed field-extraction output persistence;
- billing queue implementation;
- multi-region recovery.
