# Invoice Manager Failure Propagation — Target State

## Goal

A background-processing failure must be visible in two different forms:

1. operational detail for engineering/support investigation;
2. a safe, actionable status for the user.

## Authoritative Job State

SQL is the target-state authority for the job record. It should include at least:

```text
JobId
Status
CurrentStage
FailureCategory
UserMessage
CanRetry
LastUpdatedAt
```

Examples of user-visible state:

```json
{
  "jobId": "job-123",
  "status": "Failed",
  "stage": "TextExtraction",
  "message": "Unsupported file format.",
  "canRetry": false
}
```

or

```json
{
  "jobId": "job-456",
  "status": "Failed",
  "stage": "TextExtraction",
  "message": "Processing failed. Please try again later.",
  "canRetry": true
}
```

## Operational Failure Detail

Retain technical detail separately from browser-facing data:

```text
JobId
MessageId
Stage
AttemptCount
FailureCategory
Technical error / exception
CorrelationId
Timestamp
```

Do not expose stack traces, credentials, internal hostnames or raw dependency responses to users.

## Classification Policy

| Category | Example | Retry | DLQ | User outcome |
|---|---|---:|---:|---|
| Business validation | Unsupported file format | No | No | Explain invalid input safely. |
| Business validation | Corrupted/unreadable document | No | No | Explain document cannot be processed. |
| Retryable dependency | Timeout or temporary service failure | Yes | Only after retry limit | Explain processing could not complete safely. |
| Operational misconfiguration | Azure AI 401/403 | No normal retry | Yes | Fail safely; operations investigate. |

## Replay

Only operational DLQ messages are candidates for replay.

Replay flow:

```text
Failed
  ↓
Root cause investigated and fixed
  ↓
Replay requested
  ↓
Reprocessing
  ↓
Completed or failed again
```

Replay must happen in controlled batches with monitoring and stop conditions.
