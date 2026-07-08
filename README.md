# Staff Engineer Interview Playbook

> Building Staff Engineer intuition through first-principles learning, production architecture, coding patterns, and deliberate practice.

Personal knowledge base for Senior / Staff Backend Engineer interview preparation.

The goal is not to memorize interview answers. The goal is to understand why technologies exist, what problems they solve, what trade-offs they introduce, and how to explain them clearly in interviews.

## Start Here

For a new chat or a fresh working session, read:

- [`START_HERE.md`](START_HERE.md)
- [`project/context.md`](project/context.md)
- [`project/roadmap.md`](project/roadmap.md)
- [`project/pending.md`](project/pending.md)
- [`STAFF_INTERVIEW_SCORECARD.md`](STAFF_INTERVIEW_SCORECARD.md)

These files are the canonical context for the Interview Prep project.

## Progress

### Coding

- ✅ Two Sum — Hash Lookup
- ✅ Best Time to Buy and Sell Stock — Running Minimum
- ✅ Valid Anagram — Frequency Counting
- ✅ Product of Array Except Self — Prefix / Suffix Computation
- ✅ Valid Palindrome — Two Pointers
- ✅ Maximum Average Subarray I — Fixed-size Sliding Window
- ✅ Find Pivot Index — Prefix Sum / Running Sum
- ✅ Linked List Cycle — Fast & Slow Pointers
- ✅ Middle of the Linked List — Fast & Slow Pointers

### Backend / Security

- ✅ APIs
- ✅ ASP.NET Core Request Pipeline
- ✅ HTTP Statelessness
- ✅ Sessions
- ✅ Cookies
- ✅ Distributed Sessions / Redis
- ✅ JWT
- ✅ OAuth 2.0
- ✅ OIDC
- ✅ Token Lifecycle
- ✅ Logout, SSO and SLO
- ✅ Zero Trust identity principles
- 🟡 API Gateway / Ingress
- ⬜ SAML
- ⬜ TLS
- ⬜ DNS
- ⬜ HTTP/2 and HTTP/3
- ⬜ gRPC

### System Design / Architecture

- ✅ Invoice Manager current architecture
- ✅ Invoice Manager responsibilities
- ✅ Invoice Manager authentication and identity architecture
- ✅ Sync vs async motivation
- ✅ Async API contract: `202 Accepted`, `jobId`, `statusUrl`
- ✅ Idempotency model
- ✅ Correlation ID vs Idempotency Key
- ✅ SQL as target-state processing-state authority
- ✅ OCR artifact persistence rationale
- ✅ Queue lifecycle from first principles
- ✅ Message completion, retry release and redelivery reconciliation
- ✅ Competing consumers, stage claims and per-job ordering
- ✅ Azure Service Bus mapping for current scope
- ✅ Separate OCR and field-extraction queues
- ✅ Billing boundary after user-confirmed submission
- ⬜ Transactional messaging / outbox
- ⬜ Multi-region async DR design

## Repository Structure

```text
backend/                  Backend fundamentals and security topics
coding/                   Coding patterns and solved problems
system-design/            System design and architecture
system-design/invoice-manager/
                          Invoice Manager architecture and target-state notes
architecture/             General architecture principles
adr/                      Architecture Decision Records
daily-journal/            Daily learning logs
project/                  Roadmap, context, pending items and skills matrix
```

## Learning Philosophy

Each topic should answer:

1. Why does this exist?
2. What problem does it solve?
3. What new problems does it introduce?
4. How is it used in production?
5. How should it be explained in a Staff Engineer interview?

## Current Status

Authentication and authorization are complete for Senior / Staff interview purposes.

The current architecture focus is the Invoice Manager transition from synchronous fail-fast processing toward an asynchronous, durable workflow. Queue fundamentals, Service Bus mapping, competing consumers, queue-vs-topic, and the OCR/field-extraction topology are complete for the current scope. The next distributed-systems topic is backpressure/concurrency control.
