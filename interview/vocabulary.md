# Vocabulary

## Architecture

- Separation of Responsibilities
- Independent Scaling
- Security Boundary
- Trust Boundary
- Public Boundary
- Internal Service Boundary
- Zero Trust
- Defense in Depth
- Least Privilege
- Business Workflow Boundary
- Historical Boundary
- Logical Boundary
- Operational Context
- Processing Lifecycle State
- System of Record
- Source System
- Durable State
- Checkpointing
- Recovery

## Authentication / Authorization

- Authentication
- Authorization
- Identity Provider
- Authorization Server
- Resource Server
- Resource Owner
- Client
- App Registration
- Audience (`aud`)
- Issuer (`iss`)
- Subject (`sub`)
- Scope (`scp`)
- Role
- Claims
- ID Token
- Access Token
- Refresh Token
- Authorization Code
- Client Credentials
- On-Behalf-Of
- OIDC
- OAuth 2.0
- JWKS
- Key Rotation
- Key Identifier (`kid`)
- Bearer Token
- Token Revocation
- Single Sign-On
- Single Logout
- Public Client
- Confidential Client
- PKCE

## Distributed Systems

- Idempotency
- Idempotency Key
- Correlation ID
- Request ID
- Job ID
- Retry
- Transient Failure
- Business Validation Failure
- Operational Failure
- Exponential Backoff
- Dead Letter Queue
- Replay
- Queue Message
- Producer
- Consumer
- Temporary Ownership
- Visibility Timeout
- Ownership Renewal
- Message Completion
- Release for Retry
- Redelivery
- Durable Artifact
- Atomic Stage Claim
- Competing Consumers
- Per-job Ordering
- Queue
- Topic
- Subscription
- Peek-Lock
- Message Lock
- CompleteMessage
- AbandonMessage
- Dead-lettering
- SQL update + queue publish gap
- Transactional Outbox
- Outbox Publisher
- At-least-once Publishing
- Idempotent Consumer
- Inbox Pattern
- Durable Uniqueness
- Unique Constraint
- Duplicate Message
- Duplicate Business Processing
- Business-table Idempotency
- Message-level Deduplication
- JobStages
- Normalized Stage State
- Backpressure
- Bounded Concurrency
- In-flight Requests
- Bottleneck
- Queue Backlog
- Oldest Message Age
- Arrival Rate
- Completion Rate
- Reconciliation

## Coding

- Repeated Work
- Hash Lookup
- Running Minimum
- Frequency Counting
- Prefix Computation
- Suffix Computation
- Two Pointers
- Sliding Window
- Fixed-size Window
- Window State
- Incoming Element
- Outgoing Element
- Auxiliary Space
- Space-Time Trade-off
- Readability vs Optimization
- Fast Pointer
- Slow Pointer
- Previous Pointer
- Current Pointer
- Next Pointer
- Pointer Reversal
- Recursive Base Case
- Dummy Head
- Tail Pointer
- Linked List Merge
- Fixed Pointer Gap
- One-pass Traversal

## Definitions Learned

### Processing Artifact

Intermediate data produced by one workflow stage and needed by later stages or recovery. Example: OCR JSON output.

### Source of Truth

The authoritative store used for correctness. In the future Invoice Manager async design, SQL is the source of truth for processing state.

### Distributed Cache

A shared cache used across application replicas. Redis can reduce latency and avoid instance affinity, but should not own recovery-critical state.

### Message Ownership

A consumer has temporary exclusive responsibility for a received message. The message remains recoverable until the consumer proves successful completion or loses/releases ownership.

### Atomic Stage Claim

A conditional SQL transition that allows only one worker to move a given job stage from pending to processing. It protects a workflow stage when duplicate work messages exist.

### Redelivery Reconciliation

When a message is delivered again, the consumer checks durable SQL state and any durable output first, then performs only the work that remains missing.


### Transactional Outbox

A pattern where a service records the message it needs to publish in an Outbox table in the same database transaction as the business state change. A separate publisher later sends the message to the queue. It prevents lost publish intent but can publish duplicates, so consumers must be idempotent.

### Idempotent Consumer

A consumer that can safely receive the same logical message more than once without repeating business work incorrectly. In the Invoice Manager target design this is achieved through SQL stage checks, atomic stage claims, idempotency keys and durable-output reconciliation.

### Durable Output Before Stage Completion

A workflow stage is marked complete only after its recovery-critical output is stored durably and SQL points to it.


### Inbox Pattern

A consumer-side pattern that records received or processed messages in durable storage, usually with a unique key such as `(ConsumerName, MessageId)` or `(ConsumerName, IdempotencyKey)`. It is useful when business tables do not naturally provide enough duplicate protection.

### Durable Uniqueness

A duplicate-safety principle where durable storage enforces that only one record can exist for the same logical business operation or message. Examples include `UNIQUE(TenantId, IdempotencyKey)` for job creation and a unique Inbox key for message-level deduplication.

### Normalized Stage State

A target-state schema decision where per-stage status, attempts, owner, timestamps, errors and artifact references are stored in a `JobStages` table rather than as many columns on `Jobs`. This was introduced and accepted on Day 11.

### Dummy Head

A temporary linked-list node used to simplify list construction. It is not part of the final answer; the real result starts at `dummy.next`.


### Backpressure

A system's response when incoming work exceeds the rate at which a downstream component can safely process it. A queue can absorb a burst, but capacity still has to be managed at the constrained dependency.

### Bounded Concurrency

A limit on how many operations may run at the same time. A per-pod limit protects one process, but it is not automatically a strict global limit across all replicas.

### One-pass Traversal

A traversal that does not restart from the beginning after first computing some intermediate result. It can use more than one loop as long as the pointer continues from its current position.
