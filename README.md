# Staff Engineer Interview Playbook

> Building Staff Engineer intuition through first-principles learning, production architecture, coding patterns, and deliberate practice.

Personal knowledge base for Senior / Staff Backend Engineer interview preparation.

The goal is not to memorize interview answers. The goal is to understand **why** technologies exist, what problems they solve, what trade-offs they introduce, and how to explain them clearly in interviews.

---

## Progress

### Coding

- ✅ Two Sum — Hash Lookup
- ✅ Best Time to Buy and Sell Stock — Running Minimum
- ✅ Valid Anagram — Frequency Counting
- ✅ Product of Array Except Self — Prefix / Suffix Computation
- ✅ Valid Palindrome — Two Pointers

### Backend

- ✅ APIs
- ✅ ASP.NET Core Request Pipeline
- ✅ HTTP Statelessness
- ✅ Sessions
- ✅ Cookies
- ✅ JWT
- 🟡 API Gateway / Ingress
- ⬜ OAuth 2.0
- ⬜ OIDC
- ⬜ SAML
- ⬜ TLS
- ⬜ DNS
- ⬜ HTTP/2 and HTTP/3
- ⬜ gRPC

### System Design / Architecture

- ✅ Invoice Manager current architecture
- ✅ Component responsibilities
- ✅ Text Extractor vs Data Extractor
- ✅ API Service vs Orchestrator historical split
- ✅ Sync vs Async introduction
- 🟡 Orchestrator failure recovery
- ⬜ Idempotency
- ⬜ Retry strategy
- ⬜ Dead Letter Queues
- ⬜ Async future-state architecture

---

## Repository Structure

```text
backend/                  Backend fundamentals
coding/                   Coding patterns and solved problems
system-design/            System design and architecture
system-design/invoice-manager/
                          Invoice Manager architecture documentation
interview/                Vocabulary and mistakes
daily-journal/            Daily learning logs
project/                  Roadmap, context, pending items
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

We have completed the foundation for:

- HTTP → Sessions → Cookies → Redis → JWT
- Core JWT validation and trade-offs
- Initial coding pattern library
- Invoice Manager architecture baseline

See:

- `project/context.md`
- `project/roadmap.md`
- `project/pending.md`
- `CHANGELOG.md`
