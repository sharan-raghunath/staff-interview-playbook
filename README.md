# Staff Engineer Interview Playbook

> Building Staff Engineer intuition through first-principles learning, production architecture, coding patterns, and deliberate practice.

Personal knowledge base for Senior / Staff Backend Engineer interview preparation.

The goal is not to memorize interview answers. The goal is to understand **why** technologies exist, what problems they solve, what trade-offs they introduce, and how to explain them clearly in interviews.

---

## Start Here

For a new chat or a fresh working session, start with:

- [`START_HERE.md`](START_HERE.md)
- [`project/context.md`](project/context.md)
- [`project/roadmap.md`](project/roadmap.md)
- [`project/pending.md`](project/pending.md)
- [`STAFF_INTERVIEW_SCORECARD.md`](STAFF_INTERVIEW_SCORECARD.md)

These files are the canonical context for the Interview Prep project.

---

## Progress

### Coding

- ✅ Two Sum — Hash Lookup
- ✅ Best Time to Buy and Sell Stock — Running Minimum
- ✅ Valid Anagram — Frequency Counting
- ✅ Product of Array Except Self — Prefix / Suffix Computation
- ✅ Valid Palindrome — Two Pointers
- ✅ Maximum Average Subarray I — Fixed-size Sliding Window
- ✅ Find Pivot Index — Prefix Sum / Running Sum

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
- ✅ Text Extractor vs Data Extractor
- ✅ API Service vs Orchestrator historical split
- ✅ Sync vs Async motivation
- ✅ Async API contract: `202 Accepted`, `jobId`, `statusUrl`
- ✅ Idempotency model
- ✅ Correlation ID vs Idempotency Key
- ✅ Retry strategy
- ✅ Dead Letter Queue concept
- ✅ Async redesign Part 1: queue placement, SQL job state, OCR artifact persistence
- 🟡 End-to-end async redesign
- ⬜ Queue technology choice
- ⬜ Multi-region async DR design

---

## Repository Structure

```text
backend/                  Backend fundamentals and security topics
coding/                   Coding patterns and solved problems
system-design/            System design and architecture
system-design/invoice-manager/
                          Invoice Manager architecture documentation
architecture/             General architecture principles
adr/                      Architecture Decision Records
interview/                Vocabulary and mistakes
daily-journal/            Daily learning logs
project/                  Roadmap, context, pending items and skills matrix
```

---

## Learning Philosophy

Each topic should answer:

1. Why does this exist?
2. What problem does it solve?
3. What new problems does it introduce?
4. How is it used in production?
5. How should it be explained in a Staff Engineer interview?

---

## Current Status

The Authentication and Authorization module is now complete for Senior / Staff interview purposes.

The next major architecture focus is evolving Invoice Manager from a synchronous processing model into an asynchronous, resilient, observable workflow.
