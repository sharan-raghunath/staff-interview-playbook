# Invoice Manager

## Architecture

Akamai -> Gateway -> Traefik -> Edge API -> API Service -> Orchestrator -> Text Extractor -> Data Extractor

## Component Responsibilities

### Akamai

Acts as the internet edge for the application.

#### Responsibilities

- Web Application Firewall (WAF)
- DDoS protection
- Edge security
- Traffic filtering before requests enter Azure

---

### Azure Application Gateway

Acts as the Azure gateway before traffic enters the AKS cluster.

#### Responsibilities

- Gateway into the Kubernetes cluster
- Layer 7 routing
- Traffic forwarding to Traefik

> **TODO**
> Confirm whether TLS termination occurs at Azure Application Gateway or Traefik.

---

### Traefik (Ingress Controller)

Ingress controller within the AKS cluster.

#### Responsibilities

- Routes requests to the appropriate Kubernetes services
- Internal ingress into the cluster
- Kubernetes-native routing and load balancing

> **TODO**
> Confirm whether Traefik performs TLS termination or receives already decrypted traffic from Azure Application Gateway.

---

### Edge API

The only public-facing application API.

The Edge API acts as the boundary between external consumers and the internal Invoice Manager platform.

#### Responsibilities

- Authenticate and authorize incoming requests
- Validate request payloads
- Integrate with external source systems (Asset Finance)
- Expose public REST APIs
- Translate external requests into internal workflows
- Never expose internal services directly

> **Design Principle**
>
> If Invoice Manager needs to integrate with a different source system in the future, a new Edge API should be introduced rather than modifying the core processing services.

---

### API Service

Internal service within the AKS cluster.

This service exists primarily for historical reasons.

Originally, the Orchestrator was developed in Java by another team, while the current team developed the API Service in .NET to implement business-specific functionality.

Today, the Orchestrator has been migrated to .NET 8.

#### Current Responsibilities

- Internal business logic
- Invoice Manager application-specific functionality
- Communication with the Orchestrator

#### Future State

The API Service and Orchestrator are candidates for consolidation into a single service, as both now run on .NET 8 and have closely related responsibilities.

The preferred architecture is:

```
Edge API
        │
        ▼
Invoice Processing Service
```

instead of:

```
Edge API
        │
        ▼
API Service
        │
        ▼
Orchestrator
```

This removes an unnecessary service boundary while keeping the public-facing Edge API separate.

---

### Orchestrator

Coordinates the complete invoice processing workflow.

The Orchestrator contains workflow logic rather than business integration logic.

#### Responsibilities

- Coordinate invoice processing
- Invoke Text Extractor
- Invoke Data Extractor
- Aggregate responses
- Manage workflow sequencing
- Handle retries and orchestration logic

The Orchestrator should remain independent of any external Asset Finance system.

---

### Text Extractor (TE)

Responsible for preparing documents for machine learning.

#### Responsibilities

- PDF → Image conversion
- Image normalization
- Rotation correction
- DPI normalization
- OCR extraction
- Parallel OCR processing across pages

The Text Extractor focuses entirely on document preparation and OCR.

---

### Data Extractor (DE)

Responsible for invoice field extraction.

The Data Extractor supports multiple extraction approaches:

1. Traditional ML-based extraction
2. GenAI-based extraction

#### Responsibilities

- Load field-specific ML models
- Execute custom NER models
- Predict invoice fields
- Support GenAI-based extraction flow
- Use Semantic Kernel and skills for GenAI orchestration
- Aggregate extracted data
- Return structured prediction results

#### ML-Based Flow

The ML-based flow uses field-specific custom NER models to extract invoice fields from OCR output.

#### GenAI-Based Flow

The GenAI-based flow uses Semantic Kernel and skills to perform extraction from OCR data using GenAI.

This allows the Data Extractor to support both deterministic model-based extraction and GenAI-driven extraction depending on the use case.

#### Boundary

The Data Extractor does not own:

- Source system integration
- Public API exposure
- User-facing request validation
- End-to-end workflow orchestration

Those responsibilities belong to Edge API and the internal orchestration layer.
---

## Responsibility Boundaries

| Component | Primary Responsibility |
|-----------|------------------------|
| Akamai | Edge security |
| Azure Application Gateway | Azure gateway |
| Traefik | Kubernetes ingress |
| Edge API | Public APIs and external system integration |
| API Service | Internal business logic (candidate for future consolidation) |
| Orchestrator | Workflow orchestration |
| Text Extractor | OCR and document preprocessing |
| Data Extractor | ML / GenAI invoice field extraction |

---

## Architectural Principles

- Single responsibility per service.
- Public traffic terminates at the Edge API.
- Internal services are never exposed externally.
- Workflow orchestration is separated from machine learning inference.
- OCR preprocessing is separated from field extraction.
- Components should be independently scalable where appropriate.
- Service boundaries should represent logical responsibilities rather than historical implementation constraints.

## Design Decisions

### API Service vs Orchestrator
- API Service owns application-specific business logic.
- Orchestrator remains application-agnostic.

### Text Extractor vs Data Extractor
- OCR preprocessing separated from ML extraction.
- Independent scaling.
- Fault isolation.
