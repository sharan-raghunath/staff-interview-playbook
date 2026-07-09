# Transactional Outbox Pattern

## Status

Covered on Day 10.

## Problem

A service often needs to do two things:

```text
1. Update business state in SQL.
2. Publish a queue message so another stage can run.
```

Those are two different systems. A crash can happen between them.

Example:

```text
SQL commit succeeds
↓ crash
queue publish never happens
```

Then business state says the next stage is eligible, but no queue message exists to trigger it.

## Simple Definition

> Before I tell another system to do work, I write myself a guaranteed reminder in my own database.

## Pattern

Instead of directly publishing after the SQL update, write the publish intent into an Outbox table in the same SQL transaction as the business state change.

```text
Same SQL transaction:
- update business/workflow state
- insert outbox row representing the message to publish
```

A separate publisher later reads pending outbox rows and sends them to the queue.

## Example

When OCR completes:

```text
Same SQL transaction:
- OCR = Completed
- store OCR artifact reference
- FieldExtraction = Pending
- Outbox row = FieldExtractionRequested(JobId, OCRArtifactReference, IdempotencyKey)
```

If the service crashes after commit, SQL still contains the outbox row. The message intent is not lost.

## Outbox Publisher

```text
Find Pending outbox rows
→ send message to queue
→ mark row as Published
```

Outbox rows are usually updated to `Published` and retained for a period for audit/debugging/replay analysis. A cleanup job can later archive or delete old published rows.

## Failure Case: Duplicate Publish

The publisher can fail here:

```text
1. Send message to queue succeeds.
2. Publisher crashes before marking outbox row Published.
```

Later, the row still looks pending and may be sent again.

Therefore:

> Outbox gives at-least-once publishing, not exactly-once publishing.

Consumers must be idempotent.

## Consumer Safety

A duplicate message should not cause duplicate business work.

For a stage message such as `FieldExtractionRequested(Job 123)`, the consumer:

```text
1. checks SQL stage state;
2. if completed, acknowledges without reprocessing;
3. if processing, does not blindly duplicate work;
4. if pending, atomically claims Pending → Processing;
5. only the worker that successfully claims the stage performs work.
```

Atomic claim example:

```sql
UPDATE JobStages
SET Status = 'Processing',
    ProcessingOwner = @workerId,
    ProcessingStartedAt = @now
WHERE JobId = @jobId
  AND Stage = @stage
  AND Status = 'Pending';
```

One row updated means the worker owns the stage. Zero rows updated means another worker already claimed/completed it.

## Key Interview Line

> Outbox prevents lost messages. Idempotent consumers protect against duplicate messages.

## Deferred

This document covers the outbox pattern only. The Inbox pattern, event sourcing, Kafka, exactly-once semantics, and multi-region outbox recovery remain deferred.
