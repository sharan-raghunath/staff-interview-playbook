# Invoice Manager Overview

## Purpose

Invoice Manager is an intelligent document processing platform.

It is used to:

1. Accept invoice documents.
2. Extract OCR text.
3. Predict invoice fields using ML and/or GenAI.
4. Allow users to override predictions.
5. Submit the final reviewed result to a configured source system.

## Not a System of Record

Invoice Manager is **not** the system of record for invoice business data.

The configured source system owns:

- invoice persistence;
- invoice lifecycle;
- accounting/business state;
- downstream business processing.

Invoice Manager currently owns:

- document processing;
- prediction lifecycle while the request is being handled;
- user overrides;
- pricing-related metadata;
- operational/application metadata;
- submission orchestration.

## Target-State Processing Ownership

The proposed async architecture gives Invoice Manager responsibility for durable processing state and temporary artifacts needed to recover its own workflow. This is a target-state capability, not a claim about today’s synchronous fail-fast flow.

Examples of target-state processing information:

```text
Job ID
Current stage
Artifact reference
Failure category
Retry/replay history
```

This remains workflow state, not invoice business ownership.

## Session 7 Architecture Notes

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

Text Extractor abstracts Azure Computer Vision and Azure Document Intelligence so that Orchestrator does not need OCR-provider-specific knowledge.
