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
