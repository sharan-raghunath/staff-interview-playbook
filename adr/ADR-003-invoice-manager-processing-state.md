# ADR-003: Invoice Manager Owns Processing State, Not Invoice Business Data

## Status

Accepted.

## Context

Invoice Manager predicts fields from invoice documents, allows user overrides, and submits the result to a configured source system.

It is not the system of record for invoice business data.

## Decision

Invoice Manager should not be treated as the invoice system of record.

It should own processing lifecycle state for uploads/jobs.

## Rationale

The source system owns the actual invoice business lifecycle.

Invoice Manager owns the processing workflow required to produce predictions and submit reviewed data.

## Consequences

DR and async design should focus on:

- uploaded artifacts
- processing jobs
- workflow state
- retries
- DLQ
- user review state if applicable

not on long-term invoice business ownership.
