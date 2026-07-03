# ADR-007 — Persist OCR Output as Processing Artifact

## Status

Accepted for target-state design.

## Context

In the future asynchronous workflow, OCR may complete successfully before the Orchestrator or Data Extraction stage fails. If OCR output is not persisted, retries must rerun OCR, increasing cost, latency and dependency load.

## Decision

Persist OCR output as a temporary processing artifact.

Suggested split:

- SQL stores job state and artifact references.
- Blob Storage stores OCR JSON, page-level OCR output and extracted text.

## Rationale

Persisting OCR artifacts enables:

- Resume from OCR complete.
- Avoid repeated Azure CV / Azure DI calls.
- Lower cost.
- Lower latency.
- Better troubleshooting.

## Non-Goal

This does not make Invoice Manager the system of record for invoice business data.

Invoice Manager owns temporary processing artifacts, not invoice lifecycle ownership.

## Interview Talking Point

> Invoice Manager does not own the invoice as business data, but it should own the processing artifacts required to complete and recover its workflow.
