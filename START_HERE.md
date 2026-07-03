# START HERE — Interview Prep Context

This file is the first file to read when starting a new chat in the Interview Prep project.

## Goal

Prepare for Senior / Staff Backend Engineer interviews with a strong focus on:

- backend engineering
- distributed systems
- system design
- cloud architecture
- coding patterns
- Staff-level communication

Target companies include Atlassian, Microsoft, Adobe, SAP, Nutanix, Dassault Systèmes and Google-level opportunities.

---

## How to Work With This Repository

Use the repository as the canonical source of truth. The chat is a working session; the repository is the durable memory.

Preferred daily rhythm:

1. Backend / theory topic
2. Coding pattern and one LeetCode problem
3. System design discussion using Invoice Manager
4. Interview-quality wording
5. End-of-day repository update

Coaching style:

- Use first-principles reasoning.
- Derive concepts from the problems that created them.
- Avoid memorized definitions.
- Use Socratic questions where possible.
- Use a hint ladder for coding and do not provide algorithmic hints too early.
- Separate technical correctness from interview-quality communication.

---

## Current Learning State

### Completed Coding Problems

| Day | Problem | Pattern | Status |
|-----|---------|---------|--------|
| Day 1 | Two Sum | Hash Lookup | ✅ Complete |
| Day 2 | Best Time to Buy and Sell Stock | Running Minimum | ✅ Complete |
| Day 3 | Valid Anagram | Frequency Counting | ✅ Complete |
| Day 4 | Product of Array Except Self | Prefix / Suffix | ✅ Complete |
| Day 5 | Valid Palindrome | Two Pointers | ✅ Complete |
| Day 6 | Maximum Average Subarray I | Fixed-size Sliding Window | ✅ Complete |
| Day 7 | Find Pivot Index | Prefix Sum / Running Sum | ✅ Complete |

### Completed Backend Topics

- HTTP statelessness
- sessions
- cookies
- distributed sessions with Redis
- JWT
- claims
- `kid`
- JWKS
- key rotation
- roles vs scopes
- RBAC vs ABAC
- access token vs ID token
- refresh tokens
- refresh token rotation
- revocation
- OAuth 2.0
- Authorization Code Flow
- Implicit Flow
- Client Credentials Flow
- On-Behalf-Of Flow concept
- OIDC
- token lifecycle
- logout
- SSO and SLO
- Zero Trust identity principles

### Current System Design Focus

Invoice Manager is the primary running example.

Invoice Manager is **not** the system of record for invoice business data. It predicts invoice fields, allows the user to override predictions, and submits the final result to a configured source system. The source system owns the business invoice data.

Invoice Manager does own the **processing lifecycle state** of an upload/job.

---

## Canonical Invoice Manager Architecture

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

Important clarifications:

- Edge API and Web App are one public-facing service boundary.
- Edge API / Web App hosts the Angular app and exposes user-facing backend endpoints.
- API Service is internal.
- Orchestrator coordinates the processing workflow.
- Text Extractor handles PDF/image preprocessing and OCR.
- Data Extractor handles ML and GenAI field prediction using Semantic Kernel and skills.
- Orchestrator, TE and DE can share an App Registration if they represent one internal processing security boundary.

---

## Important Architecture Principles Derived So Far

- Design identities around **security boundaries**, not deployment units.
- One App Registration per independent protected resource/security boundary.
- Multiple tightly coupled internal services may share an identity if they form one logical protected resource.
- Propagate end-user identity only where it is required for authorization, auditing or business decisions.
- TE and DE do not need to know the end user; they should authenticate and authorize the calling service.
- Propagate operational context such as correlation ID, trace ID, tenant ID and job ID where useful.
- Correlation ID and Idempotency Key must remain separate.
- Long-running business operations should not be tied to HTTP request lifetimes.
- Same logical request should return the same logical outcome when retried with the same idempotency key.

---


## Day 7 Additions

- Reviewed service diagram relationships without reproducing proprietary visuals.
- Canonical runtime path includes Client Web / Akamai / Azure Application Gateway / Traefik / AKS.
- Identity path includes Client IdP federation into FIS IdP and Microsoft Entra.
- Text Extractor abstracts Azure Computer Vision and Azure Document Intelligence.
- Redis is documented as distributed cache, not source of truth.
- SQL is the future source of truth for processing state.
- OCR output should be persisted as a processing artifact in the future async design.
- Queue fundamentals are the next required building block before Azure Service Bus details.

## Where to Continue Next

Next major topic: **Invoice Manager async redesign**.

Continue from:

```text
Synchronous HTTP
  ↓
202 Accepted
  ↓
Job tracking
  ↓
Polling / future notification choices
  ↓
Idempotency
  ↓
Retries
  ↓
DLQ
  ↓
Queue topology and technology selection
```

Next session should cover:

- Queue fundamentals from first principles
- Producer / consumer / message ownership
- ACK / NACK
- Message lock / visibility timeout
- Retry and DLQ mechanics
- Then map those concepts to Azure Service Bus
- Finalize Invoice Manager queue placement and topology
