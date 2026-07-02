# ADR-004: Correlation ID and Idempotency Key Are Separate

## Status

Accepted.

## Context

A question came up whether an existing Correlation ID could serve as the Idempotency Key.

## Decision

Correlation ID and Idempotency Key should remain separate identifiers.

## Rationale

Correlation ID is for observability and distributed tracing.

Idempotency Key is for business correctness and duplicate detection.

A retry of the same logical business operation may have a different request/correlation trace while sharing the same idempotency key.

## Consequences

One logical upload may have:

```text
IdempotencyKey = same across retries
CorrelationId = different per execution trace
RequestId = different per HTTP request
JobId = same processing job
```

This keeps each identifier's responsibility clear.
