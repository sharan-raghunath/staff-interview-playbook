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
| Linked List Pointer Manipulation | 8.7/10 | Solved Reverse Linked List and Merge Two Sorted Lists; learned dummy-head merge pattern |
| Trees / Graphs | Not started | User expects rustiness due to long gap |
| Dynamic Programming | Not started | Future topic |

## System Design

| Area | Current Level | Evidence |
|---|---:|---|
| Component Responsibility | 9/10 | Can explain Edge API/API Service/Orchestrator/TE/DE/Billing boundaries |
| Authentication Architecture | 9/10 | Can reason about registrations and security boundaries |
| Async Motivation | 8.5/10 | Can explain why long-running HTTP is a smell |
| Idempotency | 8.5/10 | Distinguishes unique constraints for creation, atomic stage claims for existing work, and optional Inbox for message-level dedupe |
| Queue Lifecycle | 8.7/10 | Can distinguish renewal, release-for-retry, completion, lock expiry and redelivery |
| Competing Consumers / Ordering | 8.3/10 | Understands queue ownership, duplicate stage messages, SQL claims and per-job dependency |
| Azure Service Bus Mapping | 8/10 | Mapped Peek-Lock, renew, complete, abandon/expiry and DLQ to the learned model |
| Failure Classification / DLQ | 8/10 | Distinguishes business validation from operational failures and controlled replay |
| Invoice Manager Async Topology | 8.2/10 | Selected separate OCR/field-extraction queues; Billing stays outside Orchestrator |
| Transactional Outbox | 8/10 | Can explain SQL update + queue publish gap, durable publish intent, duplicate publish and idempotent consumers |

## Communication

| Area | Current Level | Notes |
|---|---:|---|
| First-principles reasoning | Strong | User asks why technologies exist before choosing products |
| Trade-off articulation | Strong | Especially clear in SQL/Blob recovery, queue isolation and service ownership |
| Concise interview answers | Improving | Can now articulate durable success before acknowledgement and per-job ordering; continue refining into 30–60 second answers |
| Coding narration | Improving | Continue explaining invariants while coding |


## Day 10 Additions

| Skill | Status | Notes |
|---|---|---|
| Reverse linked list | Learned | Iterative O(1) solution and recursive interview version covered. |
| SQL update + queue publish gap | Learned | Can explain how a stage can get stuck when SQL commits but the next message is not published. |
| Transactional Outbox | Learned | Can explain durable publish intent, publisher lifecycle, duplicate publish, and idempotent consumer protection. |
| Full staged Invoice Manager pipeline | Learned | Upload/original PDF, PDF preparation, OCR, field extraction, user review/submit, billing boundary. |


## Day 11 Additions

| Area | Status | Evidence |
|---|---|---|
| Idempotent Consumer | Learned | Can explain duplicate delivery and how durable uniqueness prevents duplicate business work. |
| Unique constraint for initial creation | Learned | Can distinguish first-time insert races from existing-stage atomic updates. |
| Inbox pattern | Learned conceptually | Understands Inbox as optional generic message-level dedupe, distinct from business-table uniqueness. |
| Normalized job/stage schema | Learned decision | Chose `JobStages` to avoid widening `Jobs` for every stage and metadata field. |
| Merge Two Sorted Lists | Learned | Solved linked-list merge and learned dummy head / `dummy.next` return pattern. |


## Day 12 Additions

| Area | Status | Evidence |
|---|---|---|
| Backpressure | Learned | Can distinguish burst absorption from downstream capacity and branch by the actual bottleneck. |
| Per-pod bounded concurrency | Learned conceptually | Understands that an async gate can cap in-flight calls in one pod and that pod count multiplies total concurrency. |
| Global concurrency | Pending | Raised but intentionally deferred for dedicated coverage. |
| Remove Nth Node From End | Learned | Derived brute-force and one-pass two-pointer solutions; implemented with dummy head. |
| Production incident investigation | Practiced | Used Job ID/Correlation ID, stage state, queue signals and metrics → traces → logs. |
| Artifact/state reconciliation | Reinforced | Can identify Blob-written/SQL-stale failure window and avoid repeating OCR. |
| KEDA and Gunicorn | Mentioned, not covered | Only production context was discussed; internals remain pending. |
