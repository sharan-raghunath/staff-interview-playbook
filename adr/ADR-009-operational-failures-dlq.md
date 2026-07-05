# ADR-009 — Operational Failures Require Investigation and Controlled DLQ Replay

## Status

Accepted for target-state design.

## Context

Some failures indicate dependency outage, configuration error, authentication/authorization defect, deployment issue or unexpected system behavior. These require technical evidence and possible replay after remediation.

## Decision

Operational failures should preserve technical context and enter the DLQ when:

- the failure is non-retryable but requires investigation, such as Azure AI 401/403; or
- a retryable failure exhausts its retry budget.

Replay must occur only after the root cause is addressed and in controlled batches.

## Rationale

Blind replay can repeat failure, obscure incident evidence and overload a recovering dependency. Controlled replay supports safe recovery and observability.

## Consequences

- Alerting and DLQ monitoring are required.
- Job state needs safe user-facing status separate from technical failure detail.
- Replay needs monitoring and stop conditions.

## Interview Talking Point

> We distinguish invalid user input from operational failure. The latter keeps diagnostic context, enters the DLQ when appropriate, and is replayed deliberately after the root cause is fixed.
