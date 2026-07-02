# Invoice Manager Architecture

## Current High-Level Architecture

```text
Akamai
  ↓
Azure Application Gateway
  ↓
Traefik
  ↓
Edge API / Web App
  ↓
API Service
  ↓
Orchestrator
  ├── Text Extractor
  └── Data Extractor
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

Open item:

- Confirm whether TLS termination occurs at Azure Application Gateway or Traefik.

---

### Azure Outbound Firewall

Controls outbound communication from the Azure environment to external systems.

Responsibilities:

- Whitelist outbound calls to external Identity Provider endpoints
- Whitelist outbound calls to configured source systems
- Control egress traffic from the application environment

This matters because Invoice Manager integrates with:

- Microsoft Entra / Identity Provider
- tenant-specific source systems

---

### Traefik

Ingress controller inside AKS.

Responsibilities:

- Kubernetes ingress routing
- Routing requests to the appropriate service
- Cluster-native load balancing
- Ingress-level middleware where configured

Open item:

- Confirm whether Traefik performs TLS termination or receives already decrypted traffic from Azure Application Gateway.

---

### Edge API / Web App

The only public-facing application boundary.

Important clarification:

> Edge API and Web App are one service boundary in Invoice Manager. Do not model Angular and Edge API as two independently deployed public services.

Responsibilities:

- Host/serve the Angular application
- Initiate user authentication with Microsoft Entra
- Maintain authenticated application session where applicable
- Expose public backend endpoints used by the web app
- Authenticate and authorize incoming requests
- Validate request payloads
- Integrate with external source systems
- Support multi-tenant configuration
- Translate external requests into internal workflows
- Ensure internal services are not directly exposed

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
- Internal service boundary between Edge API / Web App and Orchestrator
- User-aware authorization/business logic where applicable
- Communication with Orchestrator

Future consideration:

Since Orchestrator has been migrated to .NET 8, API Service and Orchestrator are candidates for consolidation if there is no longer a clear responsibility split.

---

### Orchestrator

Coordinates the invoice processing workflow.

Responsibilities:

- Coordinate processing stages
- Invoke Text Extractor
- Invoke Data Extractor
- Aggregate stage responses
- Manage workflow sequencing
- Handle orchestration-level errors
- Future candidate for persisted state/checkpointing

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

Text Extractor does not need end-user identity for business logic, but it should authenticate and authorize the calling service.

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
| Edge API / Web App | Public app/API boundary and external integration |
| API Service | Internal business logic / historical service boundary |
| Orchestrator | Workflow orchestration |
| Text Extractor | OCR and document preprocessing |
| Data Extractor | ML / GenAI invoice field extraction |

---

## Disaster Recovery

Production has a DR environment hosted in another Azure region.

The DR setup supports switch-over / auto failover.

Detailed DR design for async queues and in-flight jobs is pending future design.
