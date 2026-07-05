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
Create durable work item
  ↓
Queue
  ↓
Orchestrator
  ↓
Text Extractor / Data Extractor
  ↓
Persist processing artifacts and job progress
  ↓
User reviews predictions
  ↓
Submit result to source system
```

This is a target-state direction, not current production behavior.

## API Experience

The request returns quickly with `202 Accepted`, a `jobId` and a `statusUrl`.

The browser queries the status endpoint. The final choice between polling and any push mechanism remains open.

## Persisted State

SQL should be authoritative for:

- job identity;
- current stage and status;
- idempotency metadata;
- artifact references;
- safe user-facing failure status;
- operational failure metadata required for recovery.

Redis can cache status reads but cannot be the recovery authority.

## Queue and Consumer Boundary

The queue separates job creation from long-running execution.

Initial target direction:

```text
IM REST API → queue → Orchestrator → synchronous HTTP calls to TE/DE
```

Orchestrator owns queue-message processing because it consumes the queue message. TE/DE remain business-processing services rather than queue-aware workers in this incremental design.

## Failure Policy

- User input errors fail the job directly; they do not enter the DLQ.
- Retryable dependency errors retry with increasing delay and a maximum retry count.
- Operational/system failures requiring investigation enter the DLQ.
- DLQ replay happens only after root cause investigation and in controlled batches.

## Deferred Decisions

- Azure queue product and topology;
- queue vs topic;
- detailed notification design;
- transactional messaging;
- multi-region recovery;
- independent queue consumption by TE/DE.
