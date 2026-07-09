# Pending Items

This file tracks intentionally pending topics. A pending topic may be named here, but it must not be explained as if it has already been learned.

## Immediate Next Topics

### Messaging / Distributed Systems

- Backpressure and concurrency control.
- Detailed delayed-retry implementation.
- Deeper Inbox implementation details, only if needed beyond the Day 11 concept.
- Multi-region recovery for queued work.
- Exact PDF-preparation queue naming/topology.
- Exact PDF image artifact format/storage path.
- Detailed field-extraction result persistence.
- Exact physical schema details beyond the Day 11 `Jobs` + `JobStages` target model.
- Billing execution topology.
- Service Bus sessions: when the existing SQL stage dependency is insufficient.
- Topics/subscriptions for future broadcast lifecycle events, only if a real independent-consumer need arises.

### Backend / Networking

- SAML overview.
- TLS and certificates.
- DNS resolution.
- Reverse proxy vs load balancer vs API gateway vs ingress controller.
- L4 vs L7.
- HTTP/2 vs HTTP/3.
- gRPC.
- CORS and CSRF.

### Coding

- Variable-size Sliding Window.
- Two Sum II.
- Container With Most Water.
- Binary Search intro.
- Valid Parentheses.
- Maximum Depth of Binary Tree.
- Number of Islands.

## Invoice Manager Documentation TODOs

- Confirm TLS termination: App Gateway vs Traefik.
- Document Azure outbound firewall rules in more detail.
- Document DR failover mechanics.
- Document tenant-specific source-system configuration in detail.
- Revisit API Service + Orchestrator consolidation after deeper review.
- Document GenAI extraction flow in more detail.
- Document current timeout values and retry behavior from code/config.

## Repository TODOs

- Add Mermaid diagrams only after the corresponding architecture decisions are stable.
- Add ADRs as decisions are actually made.
- Add STAR stories after behavioral sessions begin.
- Add company-specific interview sections after the core curriculum matures.
