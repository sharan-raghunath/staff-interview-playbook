# Invoice Manager Architecture Addendum — Session 7

This document captures architectural relationships derived from a proprietary service diagram without reproducing the diagram itself.

## Runtime Entry Path

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

## Identity Path

```text
Client Identity Provider
  ↓
Federation
  ↓
FIS Identity Provider
  ↓
Microsoft Entra
  ↓
Invoice Manager
```

This supports client/customer identity federation while still allowing Invoice Manager to rely on Microsoft Entra as the application identity platform.

## Application-Owned Services

```text
IM Web / Edge API
  ↓
IM REST API
  ↓
Orchestrator
  ├── Text Extraction Service
  └── Data Extraction Service
```

Additional application-owned component:

- Billing service / pricing-related logic.

## Text Extraction Abstraction

Text Extraction is an application service owned by Invoice Manager.

It may call one or more Azure AI services:

```text
Text Extraction Service
  ├── Azure Computer Vision
  └── Azure Document Intelligence
```

The Orchestrator should not need to know whether OCR is performed by Azure Computer Vision, Azure Document Intelligence, or another future provider.

## Data Extraction

Data Extraction owns field prediction.

It supports:

- Traditional ML / custom NER extraction.
- GenAI extraction using Semantic Kernel and skills.

## Platform Dependencies

Invoice Manager uses several Azure platform services:

- Azure Kubernetes Service
- Azure SQL
- Azure Redis Cache
- Azure Key Vault
- Azure App Configuration
- Azure Application Insights
- Azure Log Analytics
- Availability Checks
- Azure Jump Box for operational access
- Azure Computer Vision
- Azure Document Intelligence

## Redis Distributed Cache

Redis is used as a distributed cache.

Architectural positioning:

- Useful for shared transient state across replicas.
- Avoids instance affinity.
- Can improve performance for frequently accessed data.
- Should not be the authoritative store for recovery-critical processing state.

Principle:

> Redis accelerates the system; it should not define correctness.

## Data Ownership

Invoice Manager is not the system of record for invoice business data.

It owns:

- Prediction output before submission.
- User overrides.
- Pricing metadata.
- Operational metadata.
- Current/future processing state.
- Temporary processing artifacts required for recovery.

The configured source system owns:

- Invoice lifecycle.
- Long-term invoice business data.
- Accounting/payment/business state.

## Current vs Target State

| Area | Current State | Target State |
|------|---------------|--------------|
| Processing | Synchronous | Asynchronous |
| Failure behavior | Fail-fast | Recoverable workflow |
| Queue | Not present | Introduced after business/job creation |
| Job state | Limited | Persisted processing lifecycle |
| OCR artifacts | Not yet canonical | Persisted as recovery artifact |
| Redis | Distributed cache | Cache only; not source of truth |
| SQL | Application persistence | Source of truth for processing state |

## Documentation Rule

Do not reproduce proprietary diagrams. Convert them into neutral architectural relationships, responsibility boundaries, and interview-safe abstractions.
