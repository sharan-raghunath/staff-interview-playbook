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
- Synchronous vs asynchronous communication introduction
- Current Invoice Manager synchronous bottlenecks

---

## Day 5

### Backend
- Distributed Sessions using Redis
- JWT fundamentals, JWKS and key rotation
- Claims, RBAC/ABAC, roles/scopes
- Access/refresh token behavior and revocation trade-offs

### Coding
- Valid Palindrome
- Two Pointers pattern

### System Design
- Current fail-fast behavior and need for recovery motivation

---

## Day 6

### Backend / Security
- OAuth 2.0 and OIDC from first principles
- Authorization Code, Implicit, Client Credentials and OBO concepts
- Token lifecycle, logout, SSO/SLO and Zero Trust identity principles

### Coding
- Maximum Average Subarray I
- Fixed-size Sliding Window

### System Design
- Async API contract using `202 Accepted`, `jobId` and `statusUrl`
- Idempotency
- Correlation ID vs Idempotency Key
- Retry and DLQ introduction

---

## Day 7

### Rapid-Fire Interview
- Browser-to-Invoice-Manager request lifecycle
- Current vs future failure handling
- Zero Trust and compromised-service blast radius
- Invoice Manager data ownership

### Architecture / Documentation
- Reviewed proprietary architecture relationships without reproducing visuals
- Added client edge path, identity federation path and platform/application boundaries
- Added Redis, Azure CV and Azure DI context

### System Design
- Async Invoice Manager redesign Part 1
- Queue placement discussion
- SQL selected as target-state processing-state authority
- OCR artifact persistence rationale

### Coding
- Find Pivot Index
- Prefix Sum / Running Sum

---

## Day 8

### System Design — Queue Fundamentals
- Derived why long-running business operations must outlive HTTP requests
- Added Job ID and durable SQL job-state model
- Derived queue as durable producer/consumer handoff
- Derived temporary message ownership and visibility timeout
- Derived ownership renewal for long-running OCR
- Clarified Orchestrator owns queue interaction in the incremental target state; TE stays queue-unaware while invoked over HTTP
- Derived stage completion rule and deferred transactional atomicity problem
- Classified business validation, retryable dependency and operational failures
- Added exponential backoff concept and maximum retry count
- Defined DLQ policy and controlled batch replay
- Defined user-facing vs technical failure representation

### Coding
- Linked List Cycle
- Fast & Slow Pointers / Floyd’s cycle detection
- HashSet brute force: O(n) time, O(n) space
- Optimized pointers: O(n) time, O(1) space

### Repository
- Added comprehensive Day 8 journal
- Added queue fundamentals as a canonical document
- Added failure-propagation design note
- Added linked-list-cycle coding note
- Added ADR-008 and ADR-009
- Updated context, roadmap, pending items, scorecard and skills matrix
- Removed premature queue-topic leakage from learned-content documents
