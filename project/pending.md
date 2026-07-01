# Pending Items

## Immediate Next Topics

### Backend

- OAuth 2.0 from first principles
- OIDC
- SAML
- Confirm real Invoice Manager token flow
- Access Token vs ID Token in actual architecture
- On-Behalf-Of flow if applicable

### Coding

- Sliding Window
- Two Sum II
- Container With Most Water
- Binary Search intro

### System Design

- Orchestrator crash recovery
- Idempotency
- Retry policies
- Dead Letter Queue
- Async Invoice Manager future state
- Progress tracking
- User notifications

---

## Invoice Manager Documentation TODOs

- Confirm TLS termination: App Gateway vs Traefik.
- Document Azure outbound firewall rules.
- Document DR failover details.
- Document tenant-specific source-system configuration.
- Document API Service + Orchestrator consolidation decision.
- Document GenAI extraction flow in more detail.
- Document KEDA scaling.
- Document Prometheus metrics.
- Document current timeout values.
- Document current retry behaviour.
- Document async future-state architecture.

---

## Repository TODOs

- Add `backend/oauth.md`
- Add `backend/oidc.md`
- Add `backend/saml.md`
- Add `backend/tls.md`
- Add `backend/cors.md`
- Add detailed coding pattern files:
  - `hash-lookup.md`
  - `running-minimum.md`
  - `frequency-counting.md`
  - `prefix-suffix.md`
- Add system-design invoice-manager:
  - `disaster-recovery.md`
  - `security.md`
  - `scalability.md`
  - `trade-offs.md`
  - `glossary.md`
- Add diagrams using Mermaid.
