# Idempotent Consumer and Inbox Pattern

## Status

Covered on Day 11.

## Problem

Queues and outbox publishers can deliver the same logical message more than once.

Examples:

```text
Outbox publisher sends message successfully
↓ crash before marking outbox row Published
Publisher sends same message again later
```

or:

```text
Consumer processes message
↓ acknowledgement outcome is ambiguous
Message is redelivered later
```

The system must make duplicate delivery safe.

## Idempotent Consumer

An idempotent consumer can receive the same logical message more than once without repeating business work incorrectly.

Key principle:

> Duplicate delivery is allowed. Duplicate business processing is not.

## Durable Uniqueness

Duplicate safety needs a durable uniqueness or claim mechanism somewhere.

There are three common cases.

## 1. New Business Operation Creation

Example:

```text
Duplicate upload request tries to create the same Invoice Manager job twice.
```

Atomic update is not enough because the row may not exist yet.

Use a database-enforced unique idempotency key.

```text
Jobs:
UNIQUE(TenantId, IdempotencyKey)
```

Behavior:

```text
First request inserts Job successfully.
Second duplicate request fails on unique constraint.
Second request reads existing Job and returns the same JobId.
```

## 2. Existing Stage Processing

Example:

```text
OCRRequested(JobId = 123) arrives twice.
```

The job already exists. Do not create the job again.

Use an atomic stage claim.

```sql
UPDATE JobStages
SET Status = 'Processing',
    ProcessingOwner = @workerId,
    StartedAt = @now,
    AttemptCount = AttemptCount + 1
WHERE JobId = @jobId
  AND StageName = 'OCR'
  AND Status = 'Pending';
```

Result:

```text
1 row updated
→ this worker owns the stage and may call TE

0 rows updated
→ stage is not pending
→ read state and reconcile; do not call TE blindly
```

## 3. Generic Message-level Deduplication

Use an Inbox table when the business table does not naturally represent the idempotent operation, or when generic consumer-level deduplication/audit is needed.

Example:

```text
InboxMessages
- ConsumerName
- MessageId
- IdempotencyKey
- MessageType
- Status
- ReceivedAt
- ProcessedAt
```

Possible uniqueness:

```text
UNIQUE(ConsumerName, MessageId)
```

or:

```text
UNIQUE(ConsumerName, IdempotencyKey)
```

The first consumer insert wins. Duplicate inserts fail and are treated as already seen or already processed.

## Inbox vs Business-table Constraint

They share the same underlying idea:

> Durable uniqueness makes duplicates safe.

But they are applied at different layers.

```text
Business-table unique constraint
→ protects the domain operation

Inbox table
→ protects message consumption at the consumer boundary
```

## Invoice Manager Application

```text
Upload/job creation
→ Jobs table with UNIQUE(TenantId, IdempotencyKey)

PDF preparation / OCR / field extraction
→ atomic stage claim on JobStages

Billing
→ unique BillingOperationKey on Billing/BillingRequests table,
   or Inbox if Billing needs generic message-level deduplication
```

## Relationship to Outbox

```text
Outbox
→ prevents lost messages

Idempotent Consumer / Inbox / unique constraints
→ prevents duplicate business work
```

## Interview Line

> I assume duplicate delivery can happen. If the message creates a new business operation, I use a database-enforced idempotency key. If it processes an existing stage, I use an atomic stage claim. If business tables do not naturally represent the operation, I use an Inbox table for message-level deduplication. The queue may deliver duplicates, but only one business operation is created or processed.
