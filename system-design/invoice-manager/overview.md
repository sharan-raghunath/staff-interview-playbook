# Invoice Manager Overview

## Purpose

Invoice Manager is an intelligent document processing platform.

It is used to:

1. Accept invoice documents.
2. Extract OCR text.
3. Predict invoice fields using ML and/or GenAI.
4. Allow users to override predictions.
5. Submit the final reviewed result to the configured source system.

## Not a System of Record

Invoice Manager is **not** the system of record for invoice business data.

The configured source system owns:

- invoice persistence
- invoice lifecycle
- accounting/business state
- downstream business processing

Invoice Manager owns:

- document processing
- prediction lifecycle
- user review workflow
- temporary/job state
- submission orchestration

## System of Record for Processing State

Invoice Manager should own processing lifecycle state.

Example:

```text
Accepted
↓
Queued
↓
OCR Completed
↓
Prediction Completed
↓
Waiting for User Review
↓
Submitted to Source System
↓
Completed
```

This is workflow state, not invoice business ownership.
