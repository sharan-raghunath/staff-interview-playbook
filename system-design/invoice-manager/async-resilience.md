# Invoice Manager Async and Resilience Design Notes

This file captures the Day 6 system design work. The final end-to-end async redesign will be completed in a future session.

## Core Problem

Long-running invoice processing is coupled to the lifetime of an HTTP request.

This creates problems:

- user waits on a loading screen
- browser/gateway timeouts
- wasted downstream processing after client disconnects
- duplicate uploads after refresh/retry
- difficult crash recovery
- expensive TE/DE work repeated unnecessarily

Staff-level summary:

> The business operation can outlive the HTTP request. Therefore, the business operation should be decoupled from the HTTP request lifecycle.

---

## Async API Contract

Instead of waiting for the final prediction response, the API should return quickly.

Example:

```http
HTTP/1.1 202 Accepted
```

Response body:

```json
{
  "jobId": "job-123",
  "statusUrl": "/jobs/job-123/status"
}
```

The client can use the `jobId` or `statusUrl` to track progress.

---

## Polling

Polling is a simple first option.

Advantages:

- simple to implement
- works through browsers/proxies
- easy to debug
- no persistent connection required

Disadvantages:

- many requests return no new information
- extra token validation
- extra API/database/cache load
- potential rate-limit pressure at scale

Example:

```text
5,000 concurrent jobs × 30 polls = 150,000 status requests
```

Future comparison needed:

- polling
- long polling
- Server-Sent Events
- WebSockets
- SignalR

---

## Idempotency

If the client retries the same logical upload, the backend should not process it twice.

The client should send an Idempotency Key per logical upload.

Example:

```http
POST /predict
Idempotency-Key: abc-123
```

If the original request is still processing:

```text
Return 202 Accepted with the same jobId/statusUrl.
```

If the original request completed:

```text
Return the completed result or result location.
```

If the same Idempotency Key is reused with a different payload:

```text
Reject with 400/409.
```

Possible stored metadata:

```text
IdempotencyKey
User/Tenant
PayloadHash or UploadId
JobId
Status
ResultLocation
CreatedAt
ExpiresAt
```

---

## Correlation ID vs Idempotency Key

Correlation ID is for observability.

Idempotency Key is for business correctness.

One logical upload may have:

```text
IdempotencyKey = same across retries
CorrelationId = different per execution trace
RequestId = different per HTTP attempt
JobId = same processing job
```

Do not use Correlation ID as Idempotency Key.

---

## Retry Strategy

Retries should be used for transient failures.

Good retry candidates:

- 408 Request Timeout
- 429 Too Many Requests, respecting `Retry-After`
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout
- temporary network failures
- connection resets

Usually do not retry:

- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- validation failures
- business rule failures

A 500 error requires classification. Some 500s are transient, others are programming bugs.

Staff-level question:

> Is this failure likely to succeed if I try again later?

---

## Retry Limits

Do not retry forever.

Use:

- max retry count
- exponential backoff
- jitter
- per-error classification
- circuit breakers where appropriate

After retries are exhausted, preserve the message/job for investigation and recovery.

---

## Dead Letter Queue

A DLQ stores work that could not be processed successfully after retries.

DLQ enables:

- investigation
- replay
- manual intervention
- automated recovery after dependency recovery

DLQ messages should include enough context:

- job ID
- idempotency key
- correlation ID
- tenant
- stage
- retry count
- failure reason
- exception details
- timestamp

---

## Operating Model

A production system should have:

- DLQ dashboard
- alerting on DLQ growth
- replay mechanism
- ability to quarantine poison messages
- ability to distinguish dependency outage from payload bug

---

## Next Design Step

The next session should turn these notes into a full end-to-end async architecture:

- choose queue technology
- define queue/topic topology
- define job state store
- define worker/orchestrator model
- define progress status model
- define DR behaviour
- define KEDA scaling model
