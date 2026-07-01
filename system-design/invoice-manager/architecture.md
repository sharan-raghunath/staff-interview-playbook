# Invoice Manager Architecture

## Current High-Level Architecture

```text
Akamai
  ↓
Azure Application Gateway
  ↓
Traefik
  ↓
Edge API
  ↓
API Service
  ↓
Orchestrator
  ↓
Text Extractor
  ↓
Data Extractor
```

---

## Component Responsibilities

### Akamai

Acts as the internet edge for the application.

Responsibilities:

- Web Application Firewall (WAF)
- DDoS protection
- Edge security
- IP whitelisting
- Traffic filtering before requests enter Azure

---

### Azure Application Gateway

Acts as the Azure gateway layer before traffic enters the AKS cluster.

Responsibilities:

- Gateway into Azure / AKS boundary
- Layer 7 routing
- Traffic forwarding to Traefik
- Gateway-level policies where configured

TODO:

- Confirm whether TLS termination occurs at Azure Application Gateway or Traefik.

---

### Azure Outbound Firewall

Controls outbound communication from the Azure environment to external systems.

Responsibilities:

- Whitelist outbound calls to external Identity Provider endpoints
- Whitelist outbound calls to configured source systems
- Control egress traffic from the application environment

This is important because Invoice Manager integrates with external systems such as:

- Identity Provider
- Asset Finance / tenant-specific source systems

---

### Traefik

Ingress controller inside AKS.

Responsibilities:

- Kubernetes ingress routing
- Routing requests to the appropriate service
- Cluster-native load balancing
- Ingress-level middleware where configured

TODO:

- Confirm whether Traefik performs TLS termination or receives already decrypted traffic from Azure Application Gateway.

---

### Edge API

The only public-facing application API.

The Edge API is the boundary between external consumers and internal Invoice Manager services.

Responsibilities:

- Authenticate incoming requests
- Authorize incoming requests
- Validate request payloads
- Integrate with external source systems
- Support multi-tenant configuration
- Translate external requests into internal workflows
- Ensure internal services are not directly exposed

Important clarification:

Invoice Manager is multi-tenant. A new Edge API is not necessarily required for every source-system integration. Each tenant can be configured to work with a different source system.

---

### API Service

Internal service within AKS.

The API Service exists partly for historical reasons.

Originally:

- Orchestrator was implemented in Java by another team.
- API Service was developed in .NET by the current team.

Current responsibilities:

- Internal Invoice Manager business logic
- Communication with Orchestrator
- Internal service boundary between Edge API and Orchestrator

Future consideration:

Since Orchestrator has been migrated to .NET 8, API Service and Orchestrator are candidates for consolidation if there is no longer a clear responsibility split.

---

### Orchestrator

Coordinates the invoice processing workflow.

Responsibilities:

- Coordinate invoice processing
- Invoke Text Extractor
- Invoke Data Extractor
- Aggregate responses
- Manage workflow sequencing
- Handle orchestration-level errors
- Future candidate for persistence/checkpointing

The Orchestrator should focus on workflow coordination, not external source-system integration.

---

### Text Extractor

Responsible for document preparation and OCR.

Responsibilities:

- PDF to image conversion
- Image normalization
- DPI normalization
- Rotation correction
- OCR extraction
- Parallel OCR processing across pages

Text Extractor exists separately because OCR preprocessing has its own CPU, memory and scaling profile.

---

### Data Extractor

Responsible for invoice field extraction.

Data Extractor supports:

1. ML-based extraction
2. GenAI-based extraction

Responsibilities:

- Load field-specific ML models
- Execute custom NER models
- Predict invoice fields
- Support GenAI-based extraction
- Use Semantic Kernel and skills for GenAI orchestration
- Aggregate extracted data
- Return structured prediction results

Data Extractor does not own:

- public API exposure
- source-system integration
- user-facing validation
- end-to-end workflow orchestration

---

## Responsibility Boundaries

| Component | Primary Responsibility |
|-----------|------------------------|
| Akamai | Edge security, WAF, DDoS, IP whitelisting |
| Azure Application Gateway | Azure gateway and L7 routing |
| Azure Outbound Firewall | Egress whitelisting |
| Traefik | Kubernetes ingress |
| Edge API | Public API boundary and external integration |
| API Service | Internal business logic / historical service boundary |
| Orchestrator | Workflow orchestration |
| Text Extractor | OCR and document preprocessing |
| Data Extractor | ML / GenAI invoice field extraction |

---

## Disaster Recovery

Production has a DR environment hosted in another Azure region.

The DR setup supports switch-over / auto failover.

More detailed DR design will be documented later in:

```text
system-design/invoice-manager/disaster-recovery.md
```

---

## Architectural Principles

- Only Edge API is public-facing.
- Internal services are not exposed externally.
- Public traffic is filtered before entering Azure.
- Outbound traffic to external systems requires firewall whitelisting.
- Workflow orchestration is separated from OCR and ML inference.
- OCR preprocessing and field extraction are separate because they have different compute profiles.
- Service boundaries should represent logical responsibilities, not only historical implementation constraints.
- Multi-tenancy should be handled through configuration where appropriate.
