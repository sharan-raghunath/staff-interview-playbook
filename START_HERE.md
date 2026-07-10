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
| 10 | Reverse Linked List | Pointer Manipulation | ✅ |
| 11 | Merge Two Sorted Lists | Linked List Merge / Dummy Head | ✅ |
| 12 | Remove Nth Node From End | Two Pointers / Dummy Head | ✅ |

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
IM REST API stores original PDF and creates durable SQL job state
  ↓
Transactional Outbox: PdfPreparationRequested
  ↓
PDF preparation / page-image generation
  ↓
durable page/image artifacts + SQL references
  ↓
Transactional Outbox: OCRRequested
  ↓
Orchestrator → TE
  ↓
Blob OCR artifact + SQL OCR-complete state
  ↓
Transactional Outbox: FieldExtractionRequested
  ↓
Orchestrator → DE
  ↓
durable prediction output + ReadyForUserReview state
  ↓
User review / override / submit
  ↓
IM REST API initiates Billing independently, using final user-confirmed data
```

SQL is authoritative for target-state job/stage state. Redis remains a distributed cache, not a correctness boundary.

## Queue and Outbox Concepts Learned on Days 8–10

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
- The SQL update + queue publish gap can lose the trigger for the next stage.
- Transactional Outbox records the next message-to-publish in the same SQL transaction as the stage-state update.
- Outbox gives at-least-once publishing; idempotent consumers protect against duplicate messages.
- Invoice Manager includes a PDF preparation / page-image generation stage before OCR.

## Day 11 Additions

- Idempotent consumer strategy is now covered.
- First-time job creation uses a database-enforced unique idempotency key, such as `UNIQUE(TenantId, IdempotencyKey)`.
- Existing stage processing uses an atomic `Pending → Processing` stage claim.
- Inbox pattern is understood as optional generic message-level deduplication when business tables are not sufficient.
- `JobStages` is now an accepted target-state schema decision for normalized stage state. This was introduced on Day 11; it was not assumed in earlier days.

## Day 12 Additions

- Backpressure and bottleneck-oriented scaling are covered.
- Per-pod bounded concurrency is understood conceptually; global concurrency remains deferred.
- LeetCode 19 was completed with brute-force and one-pass reasoning.
- The Friday mock reinforced Job ID/Correlation ID investigation, stage timelines, metrics → traces → logs, and durable-artifact reconciliation.
- TE/DE Gunicorn settings were discussed only as production context; KEDA and Gunicorn internals are not yet covered.

## Where to Continue Next

Day 13 begins in the next session.

- Restore the original daily balance by including a dedicated .NET/backend topic; `async`/`await` internals is the next proposed backend topic.
- Continue the coding roadmap after the linked-list block.
- Keep KEDA internals, Gunicorn internals, global concurrency and distributed rate limiting in **mentioned but not covered** status until dedicated sessions.
- Detailed delayed-retry mechanics, Service Bus sessions and multi-region queued-work recovery remain deferred.

Do not mark future topics as learned until covered.
