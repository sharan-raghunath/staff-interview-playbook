# ADR-015 — Idempotent Consumer Strategy

## Status

Accepted on Day 11.

## Context

The Transactional Outbox pattern prevents lost publish intent, but it can publish the same logical message more than once. Queues can also redeliver messages.

Therefore consumers must be idempotent.

## Decision

Use different durable duplicate-safety mechanisms depending on what the message represents.

### 1. Initial job creation

Use a database-enforced unique idempotency key on the business table.

```text
Jobs: UNIQUE(TenantId, IdempotencyKey)
```

Duplicate upload requests return the existing job rather than creating a second job.

### 2. Existing workflow stage processing

Use an atomic stage claim.

```text
Pending → Processing
```

Only the worker that successfully updates the stage state may perform the work.

### 3. Generic message-level deduplication

Use an Inbox table only when business tables do not naturally represent the operation, or when generic consumer-level dedupe/audit is required.

Possible uniqueness:

```text
UNIQUE(ConsumerName, MessageId)
```

or:

```text
UNIQUE(ConsumerName, IdempotencyKey)
```

## Consequences

- Duplicate queue messages do not create duplicate jobs or duplicate stage work.
- Business tables remain the preferred dedupe boundary when they naturally model the operation.
- Inbox remains available for consumers that need message-level dedupe independent of domain tables.

## Key Line

> Outbox prevents lost messages; idempotent consumers prevent duplicate business work.
