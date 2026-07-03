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
- Back Pressure

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
- Token Introspection
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
- Permanent Failure
- Exponential Backoff
- Jitter
- Dead Letter Queue
- Poison Message
- Replay
- At-least-once Processing
- Exactly-once Processing
- Eventual Consistency

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


## Day 7 Terms

### Processing Artifact
Intermediate data produced by one workflow stage and needed by later stages or recovery. Example: OCR JSON output.

### Source of Truth
The authoritative store used for correctness. In the future Invoice Manager async design, SQL is the source of truth for processing state.

### Distributed Cache
A shared cache used across application replicas. Redis can reduce latency and avoid instance affinity, but should not own recovery-critical state.

### Message Ownership
The idea that once a consumer takes work from a queue, the system must track whether that work is still in progress, completed, failed, or should be retried.

### Queue Lock / Visibility Timeout
A queue mechanism that prevents multiple consumers from processing the same message at the same time while allowing redelivery if the consumer crashes.
