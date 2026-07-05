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

## Completed Backend Topics

- HTTP statelessness;
- sessions and cookies;
- distributed sessions with Redis;
- JWT, JWKS, `kid`, claims and key rotation;
- roles/scopes and RBAC/ABAC;
- OAuth 2.0 and OIDC;
- Authorization Code, Implicit, Client Credentials and OBO concepts;
- ID token vs access token;
- refresh-token lifecycle;
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

Target-state direction discussed so far:

```text
IM REST API creates durable job state
  ↓
Queue hands work to Orchestrator
  ↓
Orchestrator calls TE/DE and updates processing state
```

SQL is the authority for target-state job/workflow state. Redis remains a distributed cache and not a correctness boundary.

## Queue Concepts Learned on Day 8

- HTTP lifetime and business-process lifetime should be independent.
- A long-running operation needs a Job ID and persistent state.
- A queue is a durable handoff between producer and consumer.
- Received work needs temporary exclusive ownership rather than immediate deletion.
- Visibility timeout makes abandoned work available again.
- Long-running consumers renew ownership before it expires.
- A consumer completes work only after the required outcome is persisted.
- Failures must be classified into user-input, retryable dependency and operational/system categories.
- Business validation failures do not enter DLQ.
- Operational failures and exhausted retry attempts can enter DLQ.
- DLQ replay follows root-cause remediation and runs in controlled batches.

## Where to Continue Next

Continue with **Azure Service Bus mapping**, using the concepts already derived:

- how Azure Service Bus represents temporary ownership and completion;
- product-level failure settlement terminology;
- queue versus topic;
- competing consumers and ordering;
- then finalize Invoice Manager queue topology.

Do not jump into transactional messaging, outbox/inbox, saga or Kafka comparison until the queue product mapping is complete.
