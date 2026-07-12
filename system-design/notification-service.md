# Notification Service — Guided System Design

Covered in Session 14 as a guided design, not a mock interview.

## Scope

A reusable multi-tenant notification platform used by Invoice Manager and other producer applications.

Supported channels:

- Email
- SMS
- Push
- Future channels such as WhatsApp

Supported behavior:

- asynchronous delivery;
- tenant-specific channel policy;
- user preferences / opt-out;
- templates;
- immediate and scheduled notifications;
- retries and permanent-failure handling;
- operational support and auditability.

## Responsibility boundaries

### Invoice Manager API Service

In the current Invoice Manager responsibility model, the API Service owns the final business representation after Data Extractor output is mapped and prediction overrides/business rules are applied.

It should emit the notification event only after:

1. final mapped fields are persisted;
2. the business status is ready for the user;
3. the notification publish intent is stored transactionally.

The Orchestrator coordinates Text Extractor and Data Extractor. It does not own the final mapped business result and therefore should not emit the final `InvoiceProcessingCompleted` notification event.

### Notification Service

Owns:

- recipient resolution;
- tenant configuration;
- user preferences;
- notification classification/criticality policy;
- channel selection;
- template selection and rendering;
- provider integration;
- retries and delivery status.

The producer remains channel-agnostic.

## Sample business event

```json
{
  "tenantId": "tenant-a",
  "eventType": "InvoiceProcessingCompleted",
  "invoiceId": "Invoice123",
  "recipientContext": {
    "customerId": "customer-456"
  },
  "templateData": {
    "invoiceNumber": "Invoice123"
  },
  "applicationLink": "/invoices/Invoice123",
  "idempotencyKey": "Invoice123:ProcessingCompleted"
}
```

Send only the data required for notification rendering. Do not send a raw storage link to the complete extracted JSON unless a genuine notification requirement needs it.

## High-level flow

```text
Invoice Manager API Service
        ↓
Transaction:
  persist final business result
  update processing status
  insert notification outbox event
        ↓
Invoice Manager outbox publisher
        ↓
Notification request queue / API boundary
        ↓
Notification Orchestrator
        ↓
resolve tenant policy, recipients, preferences, template and channels
        ↓
create one NotificationDelivery + outbox record per selected channel
        ↓
channel-specific queues
        ↓
Email / SMS / Push workers
        ↓
external providers
```

## Why one delivery record per channel

One logical notification can create independently retryable and observable channel deliveries:

```text
N123:Email
N123:SMS
N123:Push
```

This supports partial success, independent scaling, channel-specific retry policy and clear audit state.

## Reliability model

### At-least-once processing attempts

- durable broker retention/redelivery;
- retry policy for transient failures;
- delayed retries where appropriate;
- DLQ or permanent-failure path after retry exhaustion.

A DLQ preserves failed work for investigation. It does not guarantee successful delivery.

### Resilient end-to-end workflow

- transactional outbox prevents lost publication after a database commit;
- consumer inbox/deduplication uses stable message IDs and database unique constraints;
- idempotent business processing prevents duplicate effects;
- reconciliation detects uncertain or inconsistent states.

The inbox may be needed at each consumer boundary, including the notification orchestrator and channel workers.

## Failure classification

- Temporary timeout: retry with increasing delay.
- `429 Too Many Requests`: delayed retry, respecting provider guidance when available.
- Invalid request / business-permanent failure: record as permanent; do not retry indefinitely.
- Provider accepted request but response is uncertain: reuse a stable provider idempotency key when supported; otherwise record uncertain state and reconcile.
- Long provider outage: retry according to policy, then use secondary provider according to notification criticality and configured thresholds; exhaust to failure handling when both paths fail.

Circuit breaker was mentioned during the conversation but explicitly not covered and is not part of the learned design.

## Templates

Templates contain channel- and tenant-specific content with placeholders populated from `templateData`.

Template support was introduced conceptually in Session 14. Detailed template lifecycle/versioning remains future work.

## Scheduled notifications

Persist:

- `ScheduledAtUtc` for scheduler queries and execution;
- original local time when relevant;
- time-zone identifier for audit and recurring schedules;
- recurrence information where applicable;
- status.

A scheduler atomically claims a small batch of due records, creates the normal delivery/outbox records, commits quickly and lets the standard delivery pipeline continue.

Multiple scheduler instances can select the same due work unless rows are claimed atomically. If the scheduler crashes after the claim, creating the outbox record in the same transaction prevents the notification from being silently lost.

Outbox publication remains at least once, so downstream inbox/idempotency is still required.

## Observability

Carry both technical and business correlation fields:

- correlation ID;
- notification ID;
- delivery ID;
- idempotency key;
- tenant ID;
- user/customer identifier;
- invoice ID;
- event type;
- channel;
- provider;
- attempt number;
- status and error code.

Avoid raw PII in logs; use stable identifiers or masked values.

Useful metrics:

- total requests and deliveries;
- success/failure rate;
- failures grouped by channel, provider, error, tenant and event type;
- average, P95 and P99 latency;
- queue depth and oldest-message age;
- retry rate;
- DLQ depth;
- overdue scheduled notifications.

## Regional-failure boundary

Only high-level DR considerations were discussed. Detailed DR design remains pending.

Correctness-critical state that would need a recovery strategy includes:

- accepted requests and pending channel deliveries;
- scheduled notifications;
- inbox/outbox state;
- tenant configuration and preferences;
- templates;
- provider configuration and credentials;
- durable queued work.

Notification history may be pipeline-noncritical but still business-critical for audit, support, compliance or billing.

Caches should normally be reconstructable and are not automatically correctness-critical.
