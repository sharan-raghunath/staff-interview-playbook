# Project Context

This file captures the working context for the Interview Prep project so that a new chat can continue seamlessly.

## Goal

Prepare for Senior / Staff Backend Engineer interviews at companies such as:

- Atlassian
- Microsoft
- Adobe
- SAP
- Nutanix
- Dassault Systèmes
- Google-level opportunities

## Coaching Style

Use first-principles reasoning.

Do not teach isolated definitions.

For every concept, derive:

1. What problem existed?
2. Why did the previous solution fail?
3. What was invented to solve it?
4. What trade-offs did it introduce?
5. How would this be explained in a Staff interview?

## Session Structure

Preferred daily structure:

- one backend/theory topic
- one coding problem
- one system design discussion
- one interview-style question
- end-of-day repository update

## Coding Coaching

Use a Hint Ladder.

- Hint 0: no hints
- Hint 1: ask clarifying question
- Hint 2: identify bottleneck / repeated work
- Hint 3: point to pattern family
- Hint 4: explain algorithm only if needed

Avoid giving algorithmic hints too early.

The user prefers discovering the solution with guidance rather than being told the answer.

## Completed Coding Patterns

- Hash Lookup — Two Sum
- Running Minimum — Best Time to Buy and Sell Stock
- Frequency Counting — Valid Anagram
- Prefix / Suffix — Product of Array Except Self
- Two Pointers — Valid Palindrome

## Backend Progress

Completed:

- APIs
- HTTP Statelessness
- Sessions
- Cookies
- Distributed Sessions / Redis
- JWT
- Claims
- JWKS
- `kid`
- key rotation
- roles vs scopes
- RBAC vs ABAC
- refresh tokens
- JWT revocation
- JWT trade-offs

Pending:

- OAuth 2.0
- OIDC
- SAML
- TLS
- DNS
- HTTP/2
- HTTP/3
- gRPC
- CORS
- CSRF

## Invoice Manager Context

Invoice Manager is the primary running production example.

Current high-level architecture:

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

Important details:

- Akamai performs WAF, DDoS protection, edge security and IP whitelisting.
- Azure outbound firewall requires whitelisting for external calls, including Identity Provider and source systems.
- Edge API is the only public-facing API.
- The platform is multi-tenant.
- Tenants can be configured with different source systems.
- API Service and Orchestrator split is partly historical.
- Orchestrator was originally Java and later migrated to .NET 8.
- API Service and Orchestrator may be consolidated in the future if their responsibilities overlap.
- Text Extractor handles PDF to image, normalization, rotation, DPI and OCR.
- Data Extractor handles ML-based extraction and GenAI-based extraction using Semantic Kernel and skills.
- Production has DR environment in another Azure region.

## Current Architecture Gaps / TODOs

- Confirm TLS termination: Azure Application Gateway vs Traefik.
- Document DR in detail.
- Design async future state.
- Define idempotency model.
- Define retry policy.
- Define DLQ strategy.
- Learn WebSockets and SignalR internals.
- Confirm actual Angular / ASP.NET token flow.
- Confirm whether downstream token flow is On-Behalf-Of.

## User Strengths

- Strong production architecture intuition.
- Thinks naturally in responsibilities and trade-offs.
- Good at identifying repeated work in coding problems.
- Strong .NET / Azure background.
- Comfortable challenging assumptions.
- Learns well through Socratic reasoning.

## Areas to Improve

- Concise interview communication.
- Coding breadth across trees, graphs, DP, heaps, sliding window.
- Avoid relying on hints too early.
- Strengthen authentication/OAuth vocabulary.
- Build structured STAR stories.

## Repository Workflow

At end of each day:

1. User uploads zipped repository.
2. Assistant reviews current structure.
3. Assistant updates completed topics properly.
4. Assistant maintains pending/context files.
5. Assistant generates updated ZIP.
6. User pushes to GitHub.

The repository is the canonical knowledge base.
