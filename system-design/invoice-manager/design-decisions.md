# Invoice Manager Design Decisions

## Edge API as Public Boundary

### Decision

Only the Edge API is public-facing.

### Why

The Edge API protects the internal platform from external consumers.

It owns:

- authentication
- authorization
- request validation
- source-system integration
- tenant-aware request translation

Internal services should remain private implementation details.

---

## Multi-Tenant Source System Integration

### Decision

Invoice Manager supports tenant-specific source-system configuration.

### Why

The platform is multi-tenant.

Different tenants may integrate with different source systems without requiring a new Edge API for every integration.

This keeps the public API boundary stable while allowing tenant-specific behaviour through configuration.

---

## API Service vs Orchestrator

### Original Decision

API Service and Orchestrator were separate partly because of historical team and technology boundaries.

Originally:

- API Service was .NET
- Orchestrator was Java
- different teams owned the services

### Current State

Orchestrator has been migrated to .NET 8.

The historical technology boundary is weaker now.

### Future Direction

Evaluate consolidation if the API Service and Orchestrator no longer have distinct responsibilities.

Potential future architecture:

```text
Edge API
↓
Invoice Processing Service
↓
Text Extractor
↓
Data Extractor
```

Benefits:

- fewer network hops
- simpler deployment
- easier debugging
- reduced operational overhead

Caution:

- do not merge if responsibilities remain logically distinct
- preserve the Edge API as the public boundary

---

## Text Extractor vs Data Extractor

### Decision

Separate OCR/document preprocessing from field extraction.

### Why

Text Extractor owns:

- PDF to image conversion
- image normalization
- OCR extraction

Data Extractor owns:

- ML extraction
- GenAI extraction
- Semantic Kernel / skills flow
- structured prediction

The services have different CPU, memory, scaling and evolution profiles.

---

## ML and GenAI inside Data Extractor

### Decision

Data Extractor supports both ML-based and GenAI-based extraction.

### Why

Traditional ML / NER provides deterministic field extraction.

GenAI provides flexibility for complex documents.

Data Extractor abstracts both extraction methods behind one service boundary.

---

## Current Synchronous Processing

### Current Decision

Invoice processing is currently synchronous.

### Benefits

- simpler implementation
- easier request tracing
- straightforward response path
- acceptable for smaller documents

### Limitations

- long-running request risk
- gateway timeouts
- limited recovery
- poor UX for large documents
- duplicate work on retries

### Future Direction

Move long-running processing to asynchronous workflow with durable state and queues.

---

## DR Environment

### Decision

Production has a DR environment in another Azure region.

### Why

Supports regional failover and recovery.

Detailed DR behaviour is pending documentation.
