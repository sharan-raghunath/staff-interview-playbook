# Session 11 — Idempotent Consumers, Durable Uniqueness and Merge Two Sorted Lists

## Focus

Session 11 continued from Session 10's Transactional Outbox learning.

Session 10 established:

> Outbox prevents lost messages.

Session 11 focused on the matching consumer-side question:

> If outbox publishing and queue delivery can produce duplicate messages, how do we prevent duplicate business processing?

## Topics Covered

- Idempotent consumer concept.
- Durable uniqueness as the foundation for duplicate safety.
- Difference between business-table uniqueness, atomic stage claim, and Inbox pattern.
- Unique constraint for initial job creation.
- Normalized `Jobs` + `JobStages` schema decision for Invoice Manager target state.
- LeetCode 21 — Merge Two Sorted Lists.
- Mock interview: duplicate queue message handling.

---

## 1. Session 10 Reminder: Outbox Creates At-least-once Publishing

Transactional Outbox records a message-to-publish inside the same SQL transaction as the business/workflow state change.

Example:

```text
OCR artifact persisted
→ SQL transaction:
   - OCR stage = Completed
   - OCR artifact reference stored
   - FieldExtraction stage = Pending
   - Outbox row = FieldExtractionRequested
```

This prevents the next-stage trigger from being lost.

However, the outbox publisher can still send a message and crash before marking the outbox row as published.

```text
Send to queue succeeds
↓ crash
Outbox row still says Pending
↓ later
Publisher sends same message again
```

So outbox gives:

```text
At-least-once publishing
```

not exactly-once publishing.

Therefore the consumer must be idempotent.

Key line:

> Outbox prevents lost messages. Idempotent consumers prevent duplicate business work.

---

## 2. Idempotent Consumer — Simple Definition

An idempotent consumer can safely receive the same logical message more than once without repeating the business operation incorrectly.

For Invoice Manager:

```text
FieldExtractionRequested(JobId = 123, IdempotencyKey = FE-123-v1)
```

may arrive twice.

The correct result is not:

```text
Run field extraction twice and store two competing outcomes
```

The correct result is:

```text
Only one logical field-extraction operation is allowed to proceed.
Duplicate deliveries reconcile and acknowledge safely.
```

---

## 3. Important Distinction Learned Today

The user correctly clarified that there are two related but different duplicate-safety cases.

### Existing work item / stage

If the job and stage already exist, use an atomic conditional update.

Example:

```text
OCRRequested(JobId = 123)
```

The job already exists. The worker should not try to create the job again.

Instead, it should atomically claim the OCR stage:

```sql
UPDATE <stage-state-table>
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
→ this worker owns the stage
→ it may call TE

0 rows updated
→ stage is not pending
→ do not call TE blindly
→ read current state and reconcile
```

This atomic stage claim had already been learned earlier.

### New business operation creation

If the row does not exist yet, an atomic `Pending → Processing` update cannot protect the insert race.

Two duplicate upload requests can both observe that no job exists yet:

```text
Request A checks: no job exists
Request B checks: no job exists
Request A inserts
Request B inserts
```

The new Session 11 learning is:

> First-time creation needs a database-enforced unique idempotency key.

Example:

```text
UNIQUE(TenantId, IdempotencyKey)
```

on the `Jobs` table.

Behavior:

```text
First upload request inserts Job successfully
Second duplicate upload request hits duplicate-key violation
Second request reads existing Job and returns same JobId
```

Clean distinction:

```text
Creating a new business operation
→ unique constraint on idempotency key

Processing an existing stage
→ atomic Pending → Processing claim
```

---

## 4. Inbox Pattern vs Business-table Uniqueness

The Inbox pattern and unique constraints on business tables share the same underlying concept:

> Use durable storage with a uniqueness rule to make duplicates safe.

But they operate at different levels.

### Business-table uniqueness

Protects the domain operation itself.

Example:

```text
Jobs table:
UNIQUE(TenantId, IdempotencyKey)
```

Purpose:

```text
Do not create two jobs for the same upload/request.
```

This is preferred when the business table naturally represents the idempotent operation.

### Inbox table

Protects message consumption at the consumer boundary.

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

Purpose:

```text
This consumer has already received or processed this message/logical operation.
```

Useful when:

- business tables do not naturally represent the message's operation;
- a service consumes many different message types;
- generic message-level audit/deduplication is needed;
- duplicate protection should happen before business logic.

Interview line:

> Inbox and business-table unique constraints are related because both enforce durable uniqueness. The difference is where uniqueness is enforced. A business-table constraint protects the domain operation; an Inbox table protects message consumption.

---

## 5. Invoice Manager Duplicate-safety Model

### Upload / job creation

Duplicate case:

```text
Same upload request is retried twice.
```

Protection:

```text
Jobs table with UNIQUE(TenantId, IdempotencyKey)
```

Behavior:

```text
First request creates Job.
Duplicate request reads and returns the existing JobId.
```

### PDF preparation

Duplicate case:

```text
PdfPreparationRequested(JobId = 123) arrives twice.
```

Protection:

```text
Atomic claim on PDF preparation stage state in SQL.
```

Only the worker that successfully claims the stage performs PDF-to-image/page preparation.

### OCR

Duplicate case:

```text
OCRRequested(JobId = 123) arrives twice.
```

Protection:

```text
Atomic claim on OCR stage state in SQL.
```

Only the successful claimant calls TE.

If OCR is already completed, the duplicate does not call TE again; it reconciles and acknowledges.

### Field extraction

Duplicate case:

```text
FieldExtractionRequested(JobId = 123) arrives twice.
```

Protection:

```text
Atomic claim on field-extraction stage state in SQL.
```

Only the successful claimant calls DE.

If predictions are already durable and SQL says ready for review, the duplicate is acknowledged without reprocessing.

### Billing after user submission

Duplicate case:

```text
BillingRequested(JobId = 123, BillingOperationKey = X) arrives twice.
```

Protection:

```text
UNIQUE(TenantId, BillingOperationKey)
```

on a Billing/BillingRequests table, or an Inbox table if Billing needs generic message-level dedupe.

Billing remains outside Orchestrator and is triggered only after user-confirmed submission.

---

## 6. Schema Design Decision: Normalize Stage State

Earlier days covered the concept of authoritative stage state in SQL but did not decide the physical schema.

During Session 11, the user pointed out that adding every stage status as a new column on `Jobs` would make the table keep growing.

Example of the less scalable approach:

```text
Jobs
- PdfPreparationStatus
- PdfPreparationAttemptCount
- OcrStatus
- OcrAttemptCount
- FieldExtractionStatus
- FieldExtractionAttemptCount
- BillingStatus
- ...
```

This becomes wide and brittle as stages and per-stage metadata grow.

### Decision

Normalize stage state into a separate `JobStages` table for the target design.

### Proposed `Jobs` table

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

### Proposed `JobStages` table

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

Important uniqueness for current workflow:

```text
UNIQUE(JobId, StageName)
```

If future reruns/versions are needed, this can evolve to:

```text
UNIQUE(JobId, StageName, StageVersion)
```

### Stage statuses

Initial simple state machine:

```text
Pending
Processing
Completed
Failed
Skipped
Cancelled
```

Do not overcomplicate retry states initially. Use attempt count and error metadata alongside the main status.

### Interview reasoning

> Initially, stage statuses could be columns on the Jobs table. But as the workflow grows, each new stage adds status, attempt count, owner, timestamps, error details and artifact references. A normalized JobStages table gives a generic model for stage state, retries, claims, artifact references and operational reporting.

Important correction:

`JobStages` was introduced on Session 11 as a schema design decision. It was not assumed or decided during Sessions 8–10.

---

## 7. Coding — LeetCode 21: Merge Two Sorted Lists

### Requirement

Given two sorted linked lists, merge them into one sorted linked list and return the head of the merged list.

### User's initial approach

The user explained:

- start from the head of both lists;
- compare current node values;
- append the smaller node to the result;
- advance only the list whose node was selected;
- when one list is exhausted, append the remaining sorted list;
- time `O(n + m)`, space `O(1)`.

This was already the optimal approach.

### User's working solution

The first solution used `result` and `tail` pointers and special-cased the first node. It was correct and O(1) extra space.

### Cleaner dummy-head version

```csharp
public class Solution {
    public ListNode MergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;

        while (list1 != null && list2 != null)
        {
            if (list1.val <= list2.val)
            {
                tail.next = list1;
                list1 = list1.next;
            }
            else
            {
                tail.next = list2;
                list2 = list2.next;
            }

            tail = tail.next;
        }

        if (list1 != null)
        {
            tail.next = list1;
        }
        else
        {
            tail.next = list2;
        }

        return dummy.next;
    }
}
```

### Key learning

The dummy node is not part of the real answer. It is a placeholder that avoids special-casing the first append.

```text
dummy → merged list
```

So the result is:

```csharp
return dummy.next;
```

Interview line:

> I use a dummy head so that inserting the first node and inserting later nodes follow the same logic. The dummy itself is not part of the result, so I return `dummy.next`.

Complexity:

```text
Time: O(n + m)
Space: O(1)
```

---

## 8. Mock Interview — Duplicate Queue Message

Question:

> Your queue delivers the same message twice. How do you prevent duplicate business processing?

Polished answer:

> I assume duplicate delivery can happen, so the consumer must be idempotent. If the message creates a new business operation, such as an upload creating an Invoice Manager job, I use a database-enforced unique idempotency key, for example `UNIQUE(TenantId, IdempotencyKey)`. The first insert wins; a duplicate request reads the existing job instead of creating another one.

> If the message processes an existing workflow stage, such as OCR or field extraction, I use an atomic stage claim. The worker tries to move the stage from `Pending` to `Processing` in SQL. Only the worker that updates the row is allowed to call TE or DE. If the stage is already processing or completed, the duplicate message is acknowledged without repeating the business work.

> If the business tables do not naturally represent the operation, or if I need generic message-level deduplication across message types or consumers, I use an Inbox table with a unique key such as `(ConsumerName, MessageId)` or `(ConsumerName, IdempotencyKey)`.

> So the queue can deliver the same logical message more than once, but only one business operation is created or processed.

Short close:

> Outbox prevents lost messages; idempotent consumers prevent duplicate business work.

---

## 9. Session 11 Learning Notes

- The user initially applied atomic-stage-claim thinking to job creation.
- We clarified that atomic updates protect existing rows, but first-time creation requires a unique constraint because there may be no row to update yet.
- The user correctly identified that `JobStages` should not have been introduced as if it were already decided.
- We reset the language and then explicitly made `JobStages` a Session 11 schema decision.
- The user identified the normalization reason independently: otherwise every new stage adds more columns to `Jobs`.

## Session 11 Status

Completed:

- Idempotent consumer concept.
- Unique constraint for initial job creation.
- Inbox pattern distinction.
- Normalized `Jobs` + `JobStages` target schema decision.
- LeetCode 21 — Merge Two Sorted Lists.
- Mock interview on duplicate message delivery.

Next likely topics:

- Backpressure and concurrency control.
- Detailed delayed retry mechanics.
- Variable-size Sliding Window coding.
