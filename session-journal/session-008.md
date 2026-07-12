# Session 8 Journal

## Theme

Queue fundamentals from first principles, failure propagation, and the first linked-list pattern.

The session deliberately avoided starting from an Azure product. The goal was to derive the properties any durable work-handoff mechanism must provide, then leave vendor-specific mapping for a later session.

---

## System Design — Queue Fundamentals From First Principles

### 1. HTTP Request Lifetime vs Business Process Lifetime

We started by separating two lifecycles:

```text
HTTP request
```

and

```text
Invoice-processing business operation
```

A browser request normally lasts milliseconds or seconds. OCR and field prediction can take minutes. The business operation should therefore continue independently after the HTTP request has returned.

Staff-level principle:

> The lifetime of a transport request should not dictate the lifetime of a business process.

The async API contract remains:

```http
202 Accepted
```

```json
{
  "jobId": "job-123",
  "statusUrl": "/jobs/job-123/status"
}
```

---

### 2. Job Identity and Shared Persistent State

A long-running operation needs a durable identity and state separate from the original HTTP request.

Target-state responsibilities:

- REST API creates the job and returns the tracking reference.
- SQL is the authoritative persistent store for job state.
- Orchestrator updates workflow progress.
- The browser queries the status through a user-facing API; it does not need to know which worker instance processed the job.

Why SQL instead of Redis for job state:

- durability and recovery matter more than cache speed;
- retained state supports troubleshooting and support investigations;
- a cache loss must not lose the authoritative processing state;
- Redis can still accelerate frequently polled status reads, but it must not define correctness.

---

### 3. Why a Database Polling Loop Is Not Enough

A worker could poll SQL for rows in `Queued` state, but this has two weaknesses:

1. **Competing workers can pick the same work** unless SQL coordination and transactional claiming are added.
2. **Idle polling wastes CPU, IOPS and connections**, especially as worker replicas scale.

This led to the generic abstraction:

> A queue is a durable handoff mechanism between a producer and a consumer.

---

### 4. Queue Ownership and Visibility

A message must not disappear as soon as a consumer receives it. If the consumer crashes, the work must remain recoverable.

Derived lifecycle:

```text
Visible message
  ↓
Consumer receives it
  ↓
Temporarily hidden from other consumers
  ↓
Consumer proves completion → message removed
  OR
Consumer stops proving ownership → message becomes available again
```

The temporary hidden period is the **visibility timeout**.

If the visibility timeout is shorter than OCR processing, another consumer could receive the same work while the first worker is still running. That creates duplicate processing.

---

### 5. Ownership / Lease Renewal

For long-running OCR, the consumer must be able to extend its temporary ownership before the visibility timeout expires.

Conceptually:

```text
Consumer receives work
  ↓
Work remains hidden for a lease period
  ↓
Consumer is still processing
  ↓
Consumer renews lease
  ↓
Work remains hidden
```

The renewal is not a progress report. It simply states:

> I am still alive and still own this work item.

For the incremental Invoice Manager target state:

```text
Queue
  ↓
Orchestrator consumes message and owns its lease
  ↓
Orchestrator calls TE synchronously over HTTP
```

Therefore, **Orchestrator**, not the Flask Text Extractor, owns queue lease renewal and final completion. TE remains focused on OCR and Azure Computer Vision / Azure Document Intelligence integration.

A different future architecture could make TE its own queue consumer, but that was identified as a separate design choice and was not selected today.

---

### 6. Completing a Processing Stage

After successful processing, the consumer must tell the queue that the work item no longer needs delivery. Conceptually, this is an acknowledgement of successful handling; product-specific naming is deferred.

For an OCR-stage work item, the target workflow is conceptually:

```text
Orchestrator receives OCR work item
  ↓
Mark job stage as OCR in progress
  ↓
Call Text Extractor
  ↓
Persist OCR artifact reference and update job state
  ↓
Create next field-extraction work item
  ↓
Tell queue the OCR work item is successfully handled
```

Important rule:

> Do not mark a stage complete until the downstream work has completed and the result needed to advance the workflow has been persisted.

We explicitly identified, but did not solve, the atomicity gap between persisting state and creating the next work item. Transactional messaging is a future module.

---

### 7. Failure Classification

The key design refinement was that failure handling is not only "transient vs permanent." It must also distinguish user input problems from operational/system problems.

#### A. Business validation / input failure

Examples:

- unsupported file format;
- corrupted or unreadable document;
- invalid message payload.

Action:

```text
Fail job directly
Do not retry
Do not send to DLQ
Return safe user-facing reason
```

Reason: the system behaved correctly; the user supplied input that cannot be processed. This does not require operations investigation.

#### B. Retryable dependency failure

Examples:

- temporary Azure AI unavailability;
- 429 throttling;
- timeout;
- temporary network failure;
- service restart.

Action:

```text
Retry after delay
Use a maximum retry count
Use exponentially increasing delays
```

Why exponential backoff:

```text
Retry 1 → short delay
Retry 2 → longer delay
Retry 3 → longer again
```

This gives an unhealthy dependency time to recover rather than repeatedly increasing its load.

#### C. Operational non-retryable failure

Example:

```text
Text Extractor calls Azure Computer Vision / Azure Document Intelligence
↓
401 Unauthorized
```

A retry is unlikely to fix a credential, RBAC, configuration or deployment issue.

Action:

```text
Fail job
Capture technical failure detail
Move message to DLQ
Alert / investigate
```

The same applies to other failures that indicate system misconfiguration or a defect requiring intervention.

---

### 8. Failure Propagation to Users and Operations

The system needs two different representations of a failure.

#### Operational failure record

For troubleshooting and recovery, retain details such as:

```text
JobId
MessageId
Stage
AttemptCount
FailureCategory
Technical error / exception detail
CorrelationId
Timestamp
```

#### User-facing job status

The status API returns only safe, actionable information:

```json
{
  "jobId": "job-123",
  "status": "Failed",
  "stage": "TextExtraction",
  "message": "We could not process this document.",
  "canRetry": false
}
```

Raw exceptions, credentials, internal dependency names and stack traces must not be exposed to the browser.

---

### 9. Poison Messages, DLQ and Replay

A message must not retry forever. After the allowed retries are exhausted, the system needs a terminal operational path.

A **poison message** is a work item that cannot complete through the normal processing pipeline and requires investigation or recovery.

DLQ policy derived today:

```text
Business validation failure
  → fail job directly
  → no DLQ

Transient failure
  → retry with backoff
  → DLQ only if retry limit is exhausted

Operational / configuration failure (for example Azure AI 401)
  → no normal retry
  → DLQ and investigation
```

DLQ replay policy:

1. Investigate the root cause first.
2. Fix the underlying issue.
3. Replay intentionally, not blindly.
4. Replay in controlled batches.
5. Monitor success/failure and stop when the replay still fails.

A controlled replay state can be represented as:

```text
Failed
  ↓
Replay requested
  ↓
Reprocessing
  ↓
Completed or Failed again
```

---

## Coding — LeetCode 141: Linked List Cycle

### Pattern

Fast and slow pointers (Floyd's cycle detection / tortoise and hare).

### Problem Understanding

A singly linked list contains nodes with a reference to the next node. A cycle exists when some node eventually points back to a node already in the traversal; it does not have to point back to the first node.

### Brute Force

Store visited **node references** in a `HashSet<ListNode>`.

- If a node is encountered twice, return `true`.
- If traversal reaches `null`, return `false`.

Complexity:

```text
Time: O(n)
Space: O(n)
```

Important clarification: duplicate values do not prove a cycle. The repeated object reference does.

### Optimized Insight

Use two pointers:

```text
slow moves one node each iteration
fast moves two nodes each iteration
```

- No cycle: `fast` or `fast.next` eventually becomes `null`.
- Cycle: once both pointers are inside the finite cycle, fast gains one position on slow each iteration and must eventually meet it.

Complexity:

```text
Time: O(n)
Space: O(1)
```

### Final C# Shape

```csharp
public class Solution
{
    public bool HasCycle(ListNode head)
    {
        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null)
        {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast)
                return true;
        }

        return false;
    }
}
```

Key implementation lesson:

> State the complexity as `O(n)`, not `O(n/2)`, because Big-O drops constant factors.

---

## Where We Stopped

The queue lifecycle is now understood conceptually. The next session should map the concepts to Azure Service Bus without jumping ahead to unrelated distributed-systems patterns.

Next topics:

- Azure Service Bus mapping for the queue concepts already learned.
- Product-specific completion/failure settlement terminology.
- Queue vs topic, competing consumers and ordering.
- Continue Invoice Manager queue-placement and topology design.

Explicitly not covered today:

- transactional messaging;
- outbox/inbox patterns;
- saga;
- idempotent consumer implementation;
- Kafka comparison;
- product SDK code.
