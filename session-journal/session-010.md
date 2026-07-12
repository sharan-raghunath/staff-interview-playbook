# Session 10 — Reverse Linked List and Transactional Outbox

## Session Goal

Session 10 covered two areas:

1. Coding: linked-list reversal through iterative pointer manipulation, plus the recursive version and base case.
2. Distributed systems: the reliability gap between committing SQL state and publishing the next queue message, then the Transactional Outbox pattern.

The system-design anchor was the target-state Invoice Manager asynchronous pipeline.

---

## Coding — LeetCode 206: Reverse Linked List

### Problem

Given the head of a singly linked list, reverse the list and return the new head.

Example:

```text
1 → 2 → 3 → 4 → 5 → null
```

should become:

```text
5 → 4 → 3 → 2 → 1 → null
```

### Brute Force

A simple extra-memory approach:

1. Traverse the list and store nodes or values in a stack/array.
2. Rebuild links or values in reverse order.

Complexity:

```text
Time: O(n)
Space: O(n)
```

### Iterative Optimal Solution

The user directly identified the important pointer set:

```text
prev
curr
next/temp
```

Core loop:

```csharp
ListNode next = curr.next;
curr.next = prev;
prev = curr;
curr = next;
```

Reason:

- `next = curr.next` preserves the rest of the original list before the link is reversed.
- `curr.next = prev` reverses the current link.
- `prev = curr` moves the reversed-list tail forward.
- `curr = next` continues with the unreversed remainder.

Final solution:

```csharp
public class Solution {
    public ListNode ReverseList(ListNode head) {
        if(head == null || head.next == null)
        {
            return head;
        }

        ListNode curr = head;
        ListNode prev = null;

        while(curr != null)
        {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }

        return prev;
    }
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

Interview explanation:

> I keep `prev` as the already-reversed part and `curr` as the node I am processing. Before changing `curr.next`, I store the original next node so I do not lose the remainder of the list. Then I point `curr.next` backward to `prev`, move `prev` to `curr`, and move `curr` to the saved next node. When `curr` becomes null, `prev` is the new head.

### Recursive Version

The recursive version was covered for interview readiness, not as the preferred production implementation.

Base case:

```csharp
if (head == null || head.next == null)
{
    return head;
}
```

Why:

- an empty list is already reversed;
- a single-node list is already reversed;
- the single node reached at the end becomes the new head.

Recursive solution:

```csharp
public class Solution {
    public ListNode ReverseList(ListNode head) {
        if (head == null || head.next == null)
        {
            return head;
        }

        ListNode newHead = ReverseList(head.next);

        head.next.next = head;
        head.next = null;

        return newHead;
    }
}
```

Key line:

```csharp
head.next.next = head;
```

Meaning:

> After recursion reverses the rest of the list, `head.next` is the last node of that reversed portion. So `head.next.next = head` points that node back to the current head, appending the current head to the end of the reversed list.

Then:

```csharp
head.next = null;
```

is required to break the old forward link and avoid creating a cycle.

Recursive complexity:

```text
Time: O(n)
Space: O(n) due to call stack
```

Production note:

> The recursive version is useful for interviews, but the iterative version is preferred in production-grade backend code because it avoids call-stack growth and is easier to reason about for long lists.

---

## Theory — The SQL Update + Queue Publish Gap

Session 9 covered queues, acknowledgement, redelivery, competing consumers, Service Bus mapping, queue-vs-topic, and Invoice Manager queue topology.

Session 10 addressed the next reliability gap:

```text
Durable state update succeeds
↓
Service crashes before publishing next queue message
```

Invoice Manager example:

```text
1. OCR artifact is written to Blob Storage.
2. SQL is updated: OCR = Completed.
3. Orchestrator crashes before publishing FieldExtractionRequested.
```

Problem:

```text
SQL:
OCR = Completed
FieldExtraction = Pending / NotStarted

Blob:
OCR artifact exists

Queue:
No field-extraction message exists
```

The OCR artifact is not lost. The missing piece is the durable trigger/intention to start the next stage.

User impact:

> The user sees processing continue indefinitely or predictions never appear, because the workflow is stuck even though OCR actually completed.

This is not unique to OCR. It is the general problem of trying to update SQL and publish to a queue as two separate operations.

---

## Transactional Outbox Pattern

Simple definition:

> Before I tell another system to do work, I write myself a guaranteed reminder in my own database.

More formal definition:

> When a database state change requires a message to be published, write the business state change and the message-to-publish into the same database transaction. A separate publisher later reads the outbox table and sends the message to the queue.

### Why It Exists

Naive approach:

```text
1. Update SQL.
2. Publish message to queue.
```

Failure window:

```text
SQL commit succeeds
↓ crash
queue publish never happens
```

The outbox removes this lost-message gap by making the publish intent durable inside SQL.

### Outbox Flow

For OCR completion:

```text
Same SQL transaction:
- Set OCR = Completed
- Store OCR artifact reference/checksum/status
- Set FieldExtraction = Pending
- Insert Outbox row: FieldExtractionRequested(JobId, OCRArtifactReference, IdempotencyKey)
```

After commit, a background outbox publisher does:

```text
Find pending outbox row
→ send message to Service Bus
→ mark outbox row as Published
```

If the service crashes immediately after the SQL transaction commits, the outbox row remains in SQL. The publisher can recover later and send the message.

### Outbox Row Lifecycle

Outbox row initially:

```text
Id = 1001
MessageType = FieldExtractionRequested
Payload = { JobId, OCRArtifactReference, IdempotencyKey, CorrelationId }
Status = Pending
CreatedAt = ...
PublishedAt = null
```

After successful publish:

```text
Status = Published
PublishedAt = ...
```

Usually, published rows are retained for audit/debugging/replay analysis for a period, then archived or deleted by a cleanup job.

### Outbox Gives At-least-once Publishing

Failure case:

```text
1. Publisher sends message to Service Bus successfully.
2. Publisher crashes before marking outbox row as Published.
```

Later, the outbox row still appears pending and may be published again.

Therefore:

> Outbox prevents lost messages. It does not guarantee the message is published only once.

The consumer must remain idempotent.

---

## Idempotent Consumer After Outbox

If `FieldExtractionRequested(Job 123)` arrives twice, Orchestrator must not call DE twice blindly.

Consumer behavior:

```text
1. Receive FieldExtractionRequested(Job 123).
2. Check SQL stage state.
3. If FieldExtraction = Completed:
   - do not call DE;
   - complete/acknowledge the queue message.
4. If FieldExtraction = Processing:
   - another worker already owns or recently owned this stage;
   - do not call DE blindly;
   - complete or reconcile based on stale/timeout policy.
5. If FieldExtraction = Pending:
   - atomically claim the stage by moving Pending → Processing;
   - only the worker that succeeds calls DE.
```

Atomic stage claim:

```sql
UPDATE JobStages
SET Status = 'Processing',
    ProcessingOwner = @workerId,
    ProcessingStartedAt = @now
WHERE JobId = @jobId
  AND Stage = 'FieldExtraction'
  AND Status = 'Pending';
```

If one row is updated, the worker owns the stage. If zero rows are updated, the worker must not process the stage.

Key summary:

> Outbox solves lost publish. Idempotent consumers solve duplicate publish.

---

## Invoice Manager Full Staged Pipeline

A correction was made during Session 10: Invoice Manager does not jump straight from upload to OCR. There is also a document preparation / PDF-to-image stage before OCR.

Target workflow learned so far:

```text
Upload accepted
→ original PDF stored
→ PDF preparation / page-image generation
→ OCR
→ field extraction
→ user review / override
→ user submit
→ billing
```

The same reliability rule applies to every asynchronous processing stage:

> A stage is completed only after its durable output exists and SQL points to it. Any next asynchronous action is recorded in the outbox inside the same SQL transaction. The current queue message is acknowledged only after durable state is safe.

### Stage 1 — Upload Accepted → PDF Preparation Requested

When upload is accepted:

```text
Original PDF is stored durably.
```

Then SQL transaction:

```text
- Create Job record
- Store original PDF reference/checksum/status
- Set PdfPreparation = Pending
- Insert Outbox row: PdfPreparationRequested
```

Outbox publisher later sends the PDF-preparation work message.

### Stage 2 — PDF Preparation → OCR Requested

PDF preparation worker:

```text
1. Converts PDF pages to images / preparation artifacts.
2. Stores generated page images/artifacts durably.
3. SQL transaction:
   - PdfPreparation = Completed
   - store page/image artifact references, page count and metadata where needed
   - OCR = Pending
   - Insert Outbox row: OCRRequested
4. Acknowledge PDF-preparation queue message.
```

The exact image format, storage path, and service boundary were not derived and should not be invented in documentation.

### Stage 3 — OCR → Field Extraction Requested

OCR worker / Orchestrator:

```text
1. Receive OCR message.
2. Atomically claim OCR stage if needed.
3. Call TE.
4. TE returns OCR result.
5. Store OCR artifact in Blob Storage.
6. SQL transaction:
   - OCR = Completed
   - store OCR artifact reference/checksum/status
   - FieldExtraction = Pending
   - Insert Outbox row: FieldExtractionRequested
7. Acknowledge OCR queue message.
```

Important correction:

> OCR is not complete just because TE returned. OCR is complete only after the OCR artifact is durably stored and SQL points to it.

### Stage 4 — Field Extraction → Ready for User Review

Field-extraction worker / Orchestrator:

```text
1. Receive FieldExtractionRequested message.
2. Atomically claim FieldExtraction stage.
3. Call DE.
4. DE returns predictions / extracted fields.
5. Store prediction output durably.
6. SQL transaction:
   - FieldExtraction = Completed
   - store result reference and/or normalized prediction state
   - Job = ReadyForUserReview
7. Acknowledge field-extraction queue message.
```

Durable output options:

```text
Blob:
- raw DE response / large model output / full JSON artifact

SQL:
- job status
- field-extraction stage status
- normalized predicted fields needed by UI
- reference to raw artifact
- confidence/status/error metadata
```

The exact field-extraction persistence model remains to be derived later.

### Stage 5 — User Submit → Billing Requested

After predictions are shown, the user may review and override values. Billing should use the final user-confirmed state.

When the user submits:

```text
IM REST API SQL transaction:
- persist final submitted invoice/override state
- set Billing = Pending, if billing is asynchronous
- Insert Outbox row: BillingRequested
```

Billing remains outside Orchestrator. Orchestrator owns machine-driven document preparation, OCR and field extraction. IM REST API owns the user-confirmed business transition.

---

## Mock Interview Question

Question:

> Your service commits a database transaction that marks a stage completed, but crashes before publishing the next queue message. How do you recover?

Refined answer:

> The failure is that the service commits database state saying a stage is complete, but crashes before the next work message is published. In Invoice Manager, OCR may be completed and the OCR artifact may be durable, but the field-extraction message may never be sent. The job then gets stuck: SQL says OCR is complete, but nothing triggers field extraction.

> The root issue is that SQL update and queue publish are two separate operations. If we do `update database` and then `publish message`, a crash can happen between them. So the system needs a durable record of the intent to publish.

> I would use the Transactional Outbox pattern. In the same SQL transaction that marks the stage complete, I also insert an outbox row representing the message that must be published next, such as `FieldExtractionRequested`. Because both writes are in the same transaction, either both commit or both roll back. If the service crashes after the commit, the outbox row remains in SQL and a background publisher can later send the message to the queue.

> If the outbox publisher sends the message but crashes before marking the outbox row as published, it may publish the same message again. That is acceptable because the consumer must be idempotent. When the field-extraction worker receives a message, it checks SQL stage state and atomically tries to move the stage from `Pending` to `Processing`. Only the worker that succeeds performs the work. If the stage is already processing or completed, the duplicate message is acknowledged without redoing field extraction.

> Outbox prevents lost messages, and idempotent consumers protect against duplicate messages.

---

## Session 10 Completion

Completed:

- iterative reverse linked list;
- recursive reverse linked list and base case;
- SQL update + queue publish reliability gap;
- Transactional Outbox pattern;
- outbox row lifecycle;
- at-least-once publishing;
- idempotent consumer protection through atomic stage claim;
- Invoice Manager full staged pipeline including PDF preparation before OCR;
- mock interview answer.

Deferred:

- Inbox pattern;
- Service Bus sessions as a selected design;
- backpressure and concurrency limits;
- detailed delayed-retry implementation;
- exact PDF-preparation artifact format/storage path;
- exact field-extraction result persistence model;
- billing execution implementation details;
- multi-region recovery for outbox and queues.
