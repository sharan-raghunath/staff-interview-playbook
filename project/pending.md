# Pending Items

This file tracks what is intentionally pending. Completed topics should not remain here.

## Immediate Next Topics

### Backend / Networking

- SAML overview
- TLS and certificates
- DNS resolution
- Reverse Proxy vs Load Balancer vs API Gateway vs Ingress Controller
- L4 vs L7
- NLB vs ALB
- HTTP/2 vs HTTP/3
- gRPC
- CORS
- CSRF

### Coding

- Variable-size Sliding Window
- Two Sum II
- Container With Most Water
- Binary Search intro
- Valid Parentheses
- Reverse Linked List
- Maximum Depth of Binary Tree
- Number of Islands

### System Design

- Queue fundamentals from first principles
- Full Invoice Manager async redesign
- Queue technology choice: Azure Service Bus vs Kafka
- Queue vs Topic topology
- Job state table design
- Idempotent consumer design
- Retry backoff and jitter
- DLQ replay operations
- Progress tracking
- Polling vs SignalR vs WebSockets vs SSE
- KEDA scaling with queues
- Multi-region DR for in-flight queued work
- Outbox pattern
- Saga pattern

---

## Invoice Manager Documentation TODOs

- Confirm TLS termination: App Gateway vs Traefik.
- Document Azure outbound firewall rules in more operational detail.
- Document DR failover details.
- Document tenant-specific source-system configuration in detail.
- Document API Service + Orchestrator consolidation decision after deeper review.
- Document GenAI extraction flow in more detail.
- Document KEDA scaling.
- Document Prometheus metrics.
- Document current timeout values.
- Document current retry behaviour from code/config.

---

## Future Deep Dives From External Prompts

- L4 vs L7, ALB vs NLB, Kubernetes Service type LoadBalancer, Ingress and Gateway.
- Reverse Proxy vs API Gateway vs Ingress Controller vs Service Mesh.

---

## Repository TODOs

- Add diagrams using Mermaid.
- Add more detailed ADRs as architecture decisions are finalized.
- Add STAR stories after behavioral sessions begin.
- Add company-specific interview sections after core curriculum matures.
