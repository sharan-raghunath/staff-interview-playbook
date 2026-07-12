# ADR-012: Use Transactional Outbox for Asynchronous Stage Handoffs

- **Status:** Accepted
- **Date:** Session 10

## Context

The target Invoice Manager workflow has staged asynchronous processing. A stage completion often creates the need to publish another work message.

Examples:

```text
PDF preparation completed → OCRRequested
OCR completed → FieldExtractionRequested
User submitted final reviewed data → BillingRequested
```

A naive implementation would update SQL and then publish the queue message. That leaves a crash window where SQL commits but the message is never published.

## Decision

Use the Transactional Outbox pattern for stage handoffs.

Inside the same SQL transaction that records the durable workflow state, insert an outbox row describing the next message to publish.

A separate outbox publisher reads pending outbox rows, publishes messages to the relevant queue, and marks rows as published.

## Consequences

### Benefits

- prevents losing next-stage publish intent after SQL commits;
- makes publishing retryable;
- preserves an audit/debug trail of messages that were intended to be published;
- keeps SQL as the workflow correctness boundary.

### Trade-offs

- introduces an Outbox table and publisher process;
- publishing becomes at-least-once, so duplicates are possible;
- consumers must remain idempotent through SQL state checks, atomic stage claims and idempotency keys;
- published-row cleanup/retention must be managed operationally.

## Not Decided Here

- exact outbox table schema;
- publisher batch size and scheduling;
- multi-region outbox behavior;
- deeper Inbox implementation details only if needed beyond the Session 11 concept.
