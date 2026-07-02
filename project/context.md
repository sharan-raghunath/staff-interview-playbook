# Project Context

This file captures the working context for the Interview Prep project so that a new chat can continue seamlessly.

## Goal

Prepare for Senior / Staff Backend Engineer interviews at companies such as Atlassian, Microsoft, Adobe, SAP, Nutanix, Dassault Systèmes and Google-level opportunities.

## Coaching Style

Use first-principles reasoning.

Do not teach isolated definitions.

For every concept, derive:

1. What problem existed?
2. Why did the previous solution fail?
3. What was invented to solve it?
4. What trade-offs did it introduce?
5. How would this be explained in a Staff interview?

Preferred modes:

- **Learn Mode**: first-principles explanation without interview pressure.
- **Interview Mode**: concise answers, follow-up questions, communication polish.
- **Architect Mode**: trade-offs, system boundaries, operational concerns.

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
- Hint 1: ask a clarifying question
- Hint 2: identify bottleneck / repeated work
- Hint 3: point to pattern family
- Hint 4: explain algorithm only if needed

Avoid giving algorithmic hints too early. The user prefers discovering the solution with guidance rather than being told the answer.

## Completed Coding Problems

| Day | Problem | Pattern | Notes |
|-----|---------|---------|-------|
| Day 1 | Two Sum | Hash Lookup | Complement lookup, dictionary, O(n) |
| Day 2 | Best Time to Buy and Sell Stock | Running Minimum | Track minimum price so far |
| Day 3 | Valid Anagram | Frequency Counting | Count character occurrences |
| Day 4 | Product of Array Except Self | Prefix / Suffix | Reuse output array for O(1) extra space |
| Day 5 | Valid Palindrome | Two Pointers | O(n) time, O(1) space, skip non-alphanumeric |
| Day 6 | Maximum Average Subarray I | Fixed-size Sliding Window | Maintain window sum and max sum |

## Backend Progress

Completed:

- APIs
- ASP.NET Core Request Pipeline
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
- OAuth 2.0
- OIDC
- Authorization Code Flow
- Implicit Flow
- Client Credentials Flow
- On-Behalf-Of Flow concept
- ID Token vs Access Token
- token lifecycle
- logout
- SSO/SLO
- authentication best practices
- Zero Trust identity principles

Pending:

- SAML
- TLS
- DNS
- HTTP/2
- HTTP/3
- gRPC
- CORS
- CSRF
- mTLS
- DPoP / proof-of-possession tokens

## Invoice Manager Context

Invoice Manager is the primary running production example.

### Functional Purpose

Invoice Manager is an intelligent document processing platform.

It:

- accepts invoice documents
- performs OCR
- predicts invoice fields using ML and/or GenAI
- allows users to review and override predictions
- submits the final reviewed result to a configured source system

It is **not** the system of record for invoice business data.

The configured source system owns the actual invoice business lifecycle and persistence.

Invoice Manager owns **processing lifecycle state**, for example:

```text
Accepted → Queued → OCR Completed → Prediction Completed → Waiting for User Review → Submitted → Completed
```

or

```text
Accepted → Queued → Retrying → Dead Lettered
```

### Canonical Architecture

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

Important details:

- Akamai performs WAF, DDoS protection, edge security and IP whitelisting.
- Azure outbound firewall requires whitelisting for external calls, including Identity Provider and source systems.
- Edge API / Web App is the only public-facing application boundary.
- Edge API / Web App hosts the Angular app and exposes public backend endpoints.
- The platform is multi-tenant.
- Tenants can be configured with different source systems.
- API Service and Orchestrator split is partly historical.
- Orchestrator was originally Java and later migrated to .NET 8.
- API Service and Orchestrator may be consolidated in the future if their responsibilities overlap.
- Text Extractor handles PDF to image, normalization, rotation, DPI and OCR.
- Data Extractor handles ML-based extraction and GenAI-based extraction using Semantic Kernel and skills.
- Production has DR environment in another Azure region.

## Identity / Security Context

Important identity principles derived:

- One App Registration per security boundary, not necessarily per microservice.
- Edge API / Web App should have its own identity.
- API Service should have its own identity if it is an independent resource API.
- Orchestrator, TE and DE may share an identity if they form one internal Invoice Processing Engine.
- TE and DE should authenticate and authorize the calling service, but they do not need end-user identity for business logic.
- Service tokens can contain roles/scopes such as `OCR.Invoke` or `InvoiceProcessing.Internal`.
- End-user identity should be propagated only where authorization, audit or business decisions require it.
- Operational context such as correlation ID, trace ID, tenant ID, request ID and job ID should be propagated where useful.

## Current Architecture Gaps / TODOs

- Confirm TLS termination: Azure Application Gateway vs Traefik.
- Document detailed DR failover mechanics.
- Choose async messaging technology.
- Define queue topology.
- Define job state store schema.
- Define idempotent consumers.
- Define retry/backoff/jitter strategy.
- Define DLQ replay process.
- Learn WebSockets and SignalR internals before final notification design.

## User Strengths

- Strong production architecture intuition.
- Thinks naturally in responsibilities and trade-offs.
- Good at identifying repeated work in coding problems.
- Strong .NET / Azure background.
- Comfortable challenging assumptions.
- Learns well through Socratic reasoning.
- Asks Staff-level questions around security boundaries, identity propagation, idempotency and operational recovery.

## Areas to Improve

- Concise interview communication.
- Coding breadth across trees, graphs, DP, heaps, sliding window variants.
- Avoid relying on hints too early.
- Continue strengthening OAuth/OIDC vocabulary.
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
