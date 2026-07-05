# ADR-008 — Business Validation Failures Do Not Enter the DLQ

## Status

Accepted for target-state design.

## Context

Not every failed job reflects an unhealthy system. Examples such as unsupported file format, corrupted document or invalid payload are expected validation outcomes where retrying cannot improve the result.

## Decision

Business validation failures should:

- mark the job as failed;
- return a safe, actionable user message;
- avoid normal retry;
- avoid DLQ placement.

## Rationale

The system behaved correctly and no operational investigation is required. Placing these messages in the DLQ would mix normal user-input outcomes with failures that require engineering attention.

## Consequences

- Failure classification must occur before retry/DLQ decisions.
- Status API must expose a safe user message and `canRetry = false` where appropriate.

## Interview Talking Point

> A DLQ is for messages requiring operational investigation or recovery. Unsupported input is a business validation outcome, not an operational incident.
