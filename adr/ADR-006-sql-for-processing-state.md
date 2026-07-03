# ADR-006 — Use SQL as Source of Truth for Processing State

## Status

Accepted for target-state design.

## Context

Invoice Manager is moving toward asynchronous processing. The system needs a durable way to track job lifecycle state such as accepted, queued, OCR complete, prediction complete, failed, or dead-lettered.

Redis is available as a distributed cache, but cache semantics are not ideal for recovery-critical workflow state.

## Decision

Use SQL as the authoritative store for processing state.

Redis may be used as an optimization layer for frequently read status, but it must not be the correctness boundary.

## Rationale

SQL provides:

- Durability.
- Stronger consistency.
- Auditability.
- Easier troubleshooting.
- Long-lived retention.
- Better support for recovery after failures.

Redis provides:

- Low-latency cache reads.
- Shared transient state.
- Optional acceleration for polling-heavy status endpoints.

## Consequences

- Job state schema must be designed carefully.
- SQL availability becomes important for processing-state correctness.
- Redis failures should degrade performance but not cause job-state loss.

## Interview Talking Point

> Redis sounds like a modern answer, but not all cool answers are correct. For recovery-critical processing state, SQL is the source of truth; Redis is only an accelerator.
