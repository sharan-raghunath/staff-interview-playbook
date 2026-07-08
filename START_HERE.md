# START HERE — Interview Prep Context

Read this file first when beginning a new chat in the Interview Prep project.

## Goal

Prepare for Senior / Staff Backend Engineer interviews with emphasis on:

- backend engineering;
- distributed systems;
- system design;
- cloud architecture;
- coding patterns;
- Staff-level communication.

Target companies include Atlassian, Microsoft, Adobe, SAP, Nutanix, Dassault Systèmes and Google-level opportunities.

## How to Work With This Repository

The repository is the durable source of truth. The chat is a working session.

Preferred daily rhythm:

1. Backend / theory topic.
2. One coding pattern and one LeetCode problem.
3. System design using Invoice Manager.
4. Interview-quality wording.
5. End-of-day repository update.

Coaching rules:

- Use first principles before vendor/product features.
- Derive concepts from the problem that created them.
- Keep current production behavior separate from proposed target state.
- Do not document a concept as learned until it has actually been covered.
- Follow the agreed plan in order; do not cut, skip, or deviate unless explicitly requested.
- Use a hint ladder for coding; do not give the algorithm too early.

## Completed Coding Problems

| Day | Problem | Pattern | Status |
|---:|---|---|---|
| 1 | Two Sum | Hash Lookup | ✅ |
| 2 | Best Time to Buy and Sell Stock | Running Minimum | ✅ |
| 3 | Valid Anagram | Frequency Counting | ✅ |
| 4 | Product of Array Except Self | Prefix / Suffix | ✅ |
| 5 | Valid Palindrome | Two Pointers | ✅ |
| 6 | Maximum Average Subarray I | Fixed-size Sliding Window | ✅ |
| 7 | Find Pivot Index | Prefix Sum / Running Sum | ✅ |
| 8 | Linked List Cycle | Fast & Slow Pointers | ✅ |
| 9 | Middle of the Linked List | Fast & Slow Pointers | ✅ |

## Completed Backend Topics

- HTTP statelessness;
- sessions and cookies;
- distributed sessions with Redis;
- JWT, JWKS, `kid`, claims and key rotation;
- roles/scopes and RBAC/ABAC;
- OAuth 2.0 and OIDC;
- Authorization Code, Implicit, Client Credentials and OBO concepts;
- ID token vs access token;
- refresh-token lifecycle, rotation/reuse-detection principles;
- logout, SSO/SLO;
- Zero Trust identity principles.

## Canonical Invoice Manager Context

Invoice Manager predicts fields from an invoice, lets a user correct predictions, and submits the final reviewed result to the configured source system.

It is **not** the system of record for invoice business data.

Current production model:

```text
Client Web
  ↓
Akamai
  ↓
Azure Application Gateway
  ↓
Traefik
  ↓
IM Web / Edge API
  ↓
IM REST API
  ↓
Orchestrator
  ├── Text Extractor
  └── Data Extractor
```

- IM Web / Edge API is the public-facing service boundary and hosts Angular.
- Text Extractor performs document preprocessing and OCR.
- Text Extractor can use Azure Computer Vision or Azure Document Intelligence.
- Data Extractor performs ML/GenAI field prediction.
- Current processing is synchronous and fail-fast.
- Current persisted business-relevant data includes user prediction overrides used for pricing; it does not persist invoice business data as a system of record.

## Target-State Direction Learned So Far

```text
IM REST API creates durable SQL job state and PDF reference
  ↓
im-ocr-work
  ↓
Orchestrator → TE
  ↓
Blob OCR artifact + SQL OCR-complete state
  ↓
im-field-extraction-work
  ↓
Orchestrator → DE
  ↓
Prediction-ready state
  ↓
User review / override / submit
  ↓
IM REST API initiates Billing independently
```

SQL is authoritative for target-state job/stage state. Redis remains a distributed cache, not a correctness boundary.

## Queue Concepts Learned on Days 8–9

- HTTP lifetime and business-process lifetime should be independent.
- A long-running operation needs a Job ID and persistent SQL stage state.
- A queue is a durable handoff between producer and consumer.
- Received work needs temporary exclusive ownership rather than immediate deletion.
- Ownership can be renewed while a healthy long-running attempt continues.
- A consumer must distinguish renewal, release-for-retry, and completion.
- A stage is completed only after required durable result and SQL state are persisted.
- On redelivery, reconcile SQL and durable artifacts before repeating work.
- Multiple consumers safely share work through queue ownership plus atomic SQL stage claims.
- Invoice Manager needs per-job stage ordering, not global FIFO.
- OCR and field extraction use separate queues.
- Azure Service Bus mapping: Peek-Lock, lock renewal, Complete, Abandon/expiry, and DLQ.
- Business validation failures do not enter DLQ; operational failures and exhausted retry attempts can.
- DLQ replay follows root-cause remediation and runs in controlled batches.

## Where to Continue Next

Continue the coding curriculum with **Variable-size Sliding Window**.

For distributed systems, next derive **backpressure/concurrency control**, then address transactional messaging/outbox only after the problem has been stated precisely.

Do not mark those future topics as learned until covered.
