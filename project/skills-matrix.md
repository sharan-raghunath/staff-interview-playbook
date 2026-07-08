# Skills Matrix

This is a living assessment based on completed sessions.

## Backend / Security

| Area | Current Level | Evidence |
|---|---:|---|
| HTTP Statelessness | 9/10 | Can explain why sessions were invented |
| Sessions and Cookies | 9/10 | Can explain session ID vs server-side session data |
| Distributed Sessions | 8.5/10 | Understands Redis/shared-store trade-offs |
| JWT | 9/10 | Can explain signature, JWKS, `kid`, claims and revocation trade-offs |
| OAuth 2.0 | 8.5/10 | Can derive delegation, auth code, client credentials and OBO concept |
| OIDC | 8.5/10 | Can distinguish ID token from access token |
| Zero Trust Identity | 9/10 | Can reason about service identity vs user identity |

## Coding

| Pattern | Current Level | Evidence |
|---|---:|---|
| Hash Lookup | 8/10 | Solved Two Sum |
| Running Minimum | 8/10 | Solved Best Time to Buy/Sell Stock |
| Frequency Counting | 8/10 | Solved Valid Anagram |
| Prefix / Suffix | 7.5/10 | Solved Product Except Self with guidance and optimization |
| Two Pointers | 8.5/10 | Solved Valid Palindrome optimally |
| Fixed Sliding Window | 8/10 | Solved Maximum Average Subarray I; corrected indexing bug |
| Prefix Sum / Running Sum | 8/10 | Solved Find Pivot Index with O(n)/O(1) solution |
| Fast & Slow Pointers | 8.5/10 | Solved Linked List Cycle and Middle of the Linked List with O(n)/O(1) solutions |
| Trees / Graphs | Not started | User expects rustiness due to long gap |
| Dynamic Programming | Not started | Future topic |

## System Design

| Area | Current Level | Evidence |
|---|---:|---|
| Component Responsibility | 9/10 | Can explain Edge API/API Service/Orchestrator/TE/DE/Billing boundaries |
| Authentication Architecture | 9/10 | Can reason about registrations and security boundaries |
| Async Motivation | 8.5/10 | Can explain why long-running HTTP is a smell |
| Idempotency | 8/10 | Uses idempotency with durable-state reconciliation and atomic stage claims |
| Queue Lifecycle | 8.7/10 | Can distinguish renewal, release-for-retry, completion, lock expiry and redelivery |
| Competing Consumers / Ordering | 8.3/10 | Understands queue ownership, duplicate stage messages, SQL claims and per-job dependency |
| Azure Service Bus Mapping | 8/10 | Mapped Peek-Lock, renew, complete, abandon/expiry and DLQ to the learned model |
| Failure Classification / DLQ | 8/10 | Distinguishes business validation from operational failures and controlled replay |
| Invoice Manager Async Topology | 8.2/10 | Selected separate OCR/field-extraction queues; Billing stays outside Orchestrator |
| Transactional Messaging / Outbox | Not started | Explicitly deferred |

## Communication

| Area | Current Level | Notes |
|---|---:|---|
| First-principles reasoning | Strong | User asks why technologies exist before choosing products |
| Trade-off articulation | Strong | Especially clear in SQL/Blob recovery, queue isolation and service ownership |
| Concise interview answers | Improving | Can now articulate durable success before acknowledgement and per-job ordering; continue refining into 30–60 second answers |
| Coding narration | Improving | Continue explaining invariants while coding |
