# Invoice Manager Future State

## Goal

Move long-running invoice processing away from a purely synchronous request-response model while preserving a clear separation between Invoice Manager processing state and source-system invoice ownership.

## Proposed Direction So Far

```text
User
  ↓
IM Web / Edge API
  ↓
IM REST API
  ↓
Create durable job state in SQL
  ↓
OCR work queue
  ↓
Orchestrator → Text Extractor
  ↓
Persist OCR artifact and OCR-stage progress
  ↓
Field-extraction work queue
  ↓
Orchestrator → Data Extractor
  ↓
Persist prediction result and ReadyForUserReview state
  ↓
User reviews predictions / overrides values
  ↓
IM REST API persists final submission
  ↓
Billing initiated independently of Orchestrator
```

This is a target-state direction, not current production behavior.

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
- atomic stage claim state where duplicate work messages are possible.

Redis can cache status reads but cannot be the recovery authority.

## Queue and Consumer Boundary

The target work queues are:

```text
im-ocr-work
im-field-extraction-work
```

Orchestrator consumes both queues and invokes TE/DE synchronously over HTTP in the incremental design. It owns message lock renewal and settlement because it is the queue consumer. TE/DE remain business-processing services rather than queue-aware workers.

Separate queues were selected because OCR and field extraction have distinct dependencies, scaling behavior, retry pressure, and operational visibility needs.

## Stage Success and Recovery

For each stage:

```text
Receive message under temporary ownership
→ atomically claim stage where applicable
→ process work
→ persist required durable output
→ update SQL stage state
→ complete queue message
```

A redelivery first reconciles SQL state and durable output. It repeats processing only when neither proves the stage completed.

OCR is the concrete stage fully derived so far: its durable output is a Blob artifact referenced by SQL. The same stage-recovery principle applies to field extraction, but its exact persisted-result representation will be derived separately.

## Ordering

The required order is per job:

```text
OCR completed and artifact durable
→ field extraction eligible.
```

No global FIFO order across invoices is required. SQL stage state and atomic stage claims enforce workflow progression.

## Billing Boundary

Billing is not an Orchestrator stage.

Pricing depends on final user-reviewed values and overrides, so it is initiated only after the user submits through IM REST API. A dedicated Billing queue is a plausible future command boundary when billing needs asynchronous execution/retries, but its implementation is not yet designed.

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

- transactional messaging between SQL updates and next-stage publication;
- outbox/inbox patterns;
- detailed delayed-retry/backpressure design;
- Service Bus sessions as a selected ordering mechanism;
- detailed field-extraction output persistence;
- billing queue implementation;
- multi-region recovery.
