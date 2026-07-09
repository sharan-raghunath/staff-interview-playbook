# ADR-010: Use Separate OCR and Field-Extraction Queues

- **Status:** Accepted
- **Date:** Day 9

## Context

The future Invoice Manager workflow has two system-driven stages:

```text
OCR → field extraction
```

A single mixed processing queue carrying a `Stage` value would have lower configuration overhead. However, OCR and field extraction have different downstream dependencies, latency and compute profiles, retry pressure, scaling constraints, operational metrics, and failure modes.

## Decision

Use separate work queues for the two stages:

```text
im-ocr-work
im-field-extraction-work
```

Orchestrator pods consume each queue. SQL stage state and atomic stage claims enforce per-job progression and protect against duplicate messages.

## Consequences

### Benefits

- independent backlog and oldest-message-age visibility by stage;
- independent concurrency/scaling decisions;
- clearer retry, DLQ, and alerting operations;
- OCR dependency throttling does not obscure field-extraction operations;
- stage-specific operational boundaries are explicit.

### Costs

- more queue configuration and monitoring than one mixed queue;
- the next-stage publication consistency problem remains a separate deferred transactional-messaging concern.

## Not Decided Here

- exact Service Bus configuration values;
- sessions;
- detailed delayed-retry mechanism;
- deeper Inbox implementation details and multi-region message recovery.
