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


## Day 7 Architecture Notes

The following architecture relationships were captured from a proprietary diagram without reproducing the image:

```text
Client Web / Asset Finance Web
  ↓
Akamai
  ↓
Azure Application Gateway
  ↓
Traefik Ingress
  ↓
AKS-hosted Invoice Manager services
```

Application-owned services include IM Web / Edge API, IM REST API, Orchestrator, Billing, Text Extraction and Data Extraction.

Platform dependencies include Azure SQL, Azure Redis Cache, Azure Key Vault, Azure App Configuration, Application Insights, Log Analytics, Azure Computer Vision and Azure Document Intelligence.

Text Extraction abstracts OCR provider choice and can call Azure Computer Vision or Azure Document Intelligence.

Redis is used as distributed cache, but SQL should be source of truth for future recovery-critical processing state.
