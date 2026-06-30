# Design Decisions

This document captures the rationale behind the major architectural decisions made in Invoice Manager.

---

# Edge API

## Decision

Expose only the Edge API publicly.

## Why?

The Edge API forms the boundary between external consumers and the Invoice Manager platform.

It is responsible for:

- Authentication
- Authorization
- Request validation
- Integration with external Asset Finance systems
- Translating external requests into internal workflows

Internal services should never be exposed directly to external consumers.

## Future Considerations

If Invoice Manager needs to support another source system, introduce another Edge API rather than modifying the core processing services.

---

# API Service vs Orchestrator

## Original Decision

The API Service and Orchestrator were separated because they were owned by different teams.

- API Service was developed by the current team using .NET.
- Orchestrator was developed by another team using Java.

The API Service contained Invoice Manager-specific business logic while the Orchestrator focused on workflow coordination.

## Current State

The Orchestrator has now been migrated to .NET 8.

The original technology boundary no longer exists.

## Future Direction

Evaluate consolidating the API Service and Orchestrator into a single internal Invoice Processing Service.

The preferred architecture becomes:

```
Edge API
        │
        ▼
Invoice Processing Service
        │
        ▼
Text Extractor
        │
        ▼
Data Extractor
```

Benefits include:

- Fewer network hops
- Simpler deployment
- Easier debugging
- Reduced operational complexity

The Edge API should remain separate because it represents a logical architectural boundary.

---

# Text Extractor vs Data Extractor

## Decision

Separate OCR from field extraction.

## Why?

Although Azure OCR services can process PDFs directly, the machine learning models were trained using normalized images.

The Text Extractor performs:

- PDF to image conversion
- Image normalization
- Rotation correction
- DPI normalization
- OCR extraction

The Data Extractor performs:

- Machine learning inference
- GenAI inference
- Field prediction

Separating these responsibilities allows each service to evolve independently.

## Benefits

- Independent scaling
- Independent deployments
- Fault isolation
- Clear ownership
- Easier model evolution

---

# ML vs GenAI Extraction

## Decision

Support both traditional ML and GenAI within the Data Extractor.

## Why?

Different extraction techniques have different strengths.

Traditional ML provides deterministic field extraction using custom NER models.

GenAI provides greater flexibility for complex documents.

The Data Extractor abstracts these implementations behind a single service boundary.

---

# Synchronous Processing

## Current Decision

Invoice processing is currently synchronous.

The entire workflow completes before returning a response.

## Why?

- Simpler implementation
- Easier debugging
- Immediate response for smaller invoices

## Known Limitations

- Long-running requests
- Gateway timeout constraints
- User waits for the entire workflow
- Reduced resilience

## Future Direction

Move to an asynchronous architecture by introducing a queue after the orchestration layer.

This allows:

- Immediate acknowledgement to the client
- Background processing
- Retry support
- Better scalability
- Event-driven workflows

This future design is documented separately in `future-state.md`.

---

# Public vs Internal Services

## Decision

Only the Edge API is publicly accessible.

Everything else remains internal to the AKS cluster.

## Why?

- Smaller attack surface
- Stronger security boundaries
- Internal contracts remain private
- Independent evolution of internal services
- Easier API versioning

This follows the Backend-for-Frontend (BFF) pattern, where the Edge API represents the public boundary and internal services remain implementation details.