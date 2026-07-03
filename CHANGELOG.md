# Changelog

## Day 1

### Coding
- Two Sum
- Hash Lookup pattern

### Backend
- Why APIs exist

### System Design
- URL Shortener introduction

---

## Day 2

### Coding
- Best Time to Buy and Sell Stock
- Running Minimum pattern

### Backend
- ASP.NET Core Request Pipeline
- JWT validation introduction
- Zero Trust

### Production Architecture
- Initial API Service vs Orchestrator discussion

---

## Day 3

### Coding
- Valid Anagram
- Frequency Counting pattern

### Backend
- Stack vs Heap introduction
- Arrays as reference types
- Character arithmetic (`c - 'a'`)

### Production Architecture
- Text Extractor vs Data Extractor
- Gateway responsibilities
- Responsibilities vs Components

---

## Day 4

### Coding
- Product of Array Except Self
- Prefix / Suffix Computation
- Space optimization by reusing the output array

### Backend
- Why HTTP is stateless
- Why Sessions were invented
- How Cookies transport Session IDs
- Browser vs Angular vs ASP.NET responsibilities

### Production Architecture
- Synchronous vs Asynchronous communication introduction
- Current Invoice Manager synchronous bottlenecks
- WebSockets / SignalR identified as future learning topics

---

## Day 5

### Backend
- Distributed Sessions using Redis
- JWT fundamentals
- JWT structure: Header, Payload, Signature
- Public / private key signing
- `kid`
- JWKS
- Key rotation
- Important claims: `iss`, `aud`, `sub`, `oid`, `tid`, `exp`, `nbf`, `iat`, `scp`, `roles`
- RBAC vs ABAC
- Roles vs Scopes
- Access Token vs ID Token
- Refresh Tokens
- Refresh Token rotation
- JWT revocation
- JWT trade-offs

### Coding
- Valid Palindrome
- Two Pointers pattern
- O(n) time / O(1) space solution
- Readability vs compact code discussion

### System Design
- Orchestrator crash scenario
- Current fail-fast behaviour
- Need for recovery/checkpointing at orchestration boundary
- Future discussion planned for idempotency, retries, DLQ and async processing

### Repository
- Added project context
- Added roadmap
- Added pending items
- Updated Invoice Manager architecture and responsibility split

---

## Day 6

### Backend / Security
- OAuth 2.0 from first principles
- Password sharing problem before OAuth
- Delegated authorization
- Consent
- Client registration
- Redirect URI validation
- Authorization Code Flow
- Why Implicit Flow is being replaced
- PKCE concept and public clients
- Clarified that Invoice Manager migration is primarily from Implicit Flow to Authorization Code Flow, not necessarily PKCE
- Client Credentials Flow
- On-Behalf-Of Flow concept
- OIDC
- ID Token vs Access Token
- Token lifecycle
- Logout
- SSO and SLO
- Authentication and authorization best practices
- User identity vs service identity
- One App Registration per security boundary

### Coding
- Maximum Average Subarray I
- Fixed-size Sliding Window pattern
- Derived O(n × k) brute force
- Optimized to O(n) using window state
- Fixed indexing bug: outgoing element is `nums[i - k]`
- Key insight: maximizing average is equivalent to maximizing sum because `k` is constant

### System Design
- Reframed Invoice Manager as an intelligent document processing platform, not a system of record for invoice data
- Clarified Invoice Manager owns processing lifecycle state, not invoice business data
- Identified problem with coupling long-running processing to HTTP request lifetime
- Proposed async API contract using `202 Accepted`, `jobId` and `statusUrl`
- Compared polling trade-offs
- Introduced idempotency
- Distinguished Correlation ID from Idempotency Key
- Discussed retry strategy
- Distinguished transient and permanent failures
- Introduced Dead Letter Queue for investigation and recovery

### Architecture Principles
- Identities should follow security boundaries, not deployment units
- TE and DE should authenticate and authorize service callers but do not need end-user identity
- Operational context and business identity are separate
- Every identifier should have a single responsibility

### Repository
- Created v1.0 documentation baseline
- Added START_HERE context file
- Added authentication, OAuth, OIDC, coding and system-design documentation
- Added ADRs and architecture principles


## Day 7

### Rapid-Fire Interview
- Browser to Invoice Manager request lifecycle
- Current vs future failure handling
- Zero Trust and compromised-service blast radius
- Data ownership for Invoice Manager
- Edge API compromise and limitations of Zero Trust

### Architecture / Documentation
- Reviewed proprietary architecture diagram without reproducing it
- Captured architecture relationships in neutral documentation
- Added Client Web / Akamai / App Gateway / Traefik / AKS runtime path
- Added identity federation path: Client IdP → Federation → FIS IdP → Microsoft Entra
- Added Azure Redis Cache as distributed cache
- Added Azure Computer Vision and Azure Document Intelligence behind Text Extraction abstraction
- Added platform vs application-owned service boundaries

### System Design
- Async Invoice Manager redesign Part 1
- Queue placement discussion based on responsibilities
- SQL vs Redis for job state
- SQL selected as source of truth for processing state
- Redis positioned as cache only
- Future workflow states: accepted, queued, OCR in progress, OCR complete, inference in progress, prediction complete
- OCR output persistence as a future processing artifact
- Decided to study queue fundamentals before Azure Service Bus details

### Coding
- Find Pivot Index
- Prefix Sum / Running Sum pattern
- Derived O(n²) brute force
- Optimized to O(n) using total sum and running left sum
- Final C# solution accepted conceptually

### Repository
- Added Day 7 journal
- Added Find Pivot Index coding note
- Added async redesign Part 1 document
- Added queue fundamentals placeholder
- Added ADR-006 for SQL as processing-state source of truth
- Added ADR-007 for OCR artifact persistence
- Added Staff Interview Scorecard
