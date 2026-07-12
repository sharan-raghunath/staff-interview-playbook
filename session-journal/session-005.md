# Session 5

## Backend

### Topics Covered

- Distributed sessions
- Redis as shared session store
- JWT
- Public/private key signing
- `kid`
- JWKS
- Key rotation
- Claims
- RBAC vs ABAC
- Roles vs Scopes
- ID Token vs Access Token
- Refresh Tokens
- Refresh Token rotation
- JWT revocation
- JWT trade-offs

### Key Mental Model

```text
HTTP
↓
Stateless
↓
Sessions
↓
Cookies
↓
Multiple servers
↓
Redis/shared session store
↓
JWT
```

JWT exists because shared session stores solve horizontal scaling but put Redis/database in the hot path for every authenticated request.

JWT allows local token validation but makes immediate revocation harder.

---

## Coding

### Problem

Valid Palindrome

### Pattern

Two Pointers

### Key Insight

Avoid building a normalized reversed copy.

Use one pointer from the left and one from the right.

Skip non-alphanumeric characters and compare valid characters case-insensitively.

### Complexity

- Time: O(n)
- Space: O(1)

### Interview Lesson

Optimal Big-O is not the only goal.

Readable pointer logic is better than overly compact code.

---

## System Design

### Scenario

A user uploads a large invoice.

Text Extractor completes OCR.

Orchestrator crashes before invoking Data Extractor.

### Current Behaviour

- Request fails.
- User receives 500-style error.
- No durable recovery.
- Retrying may restart the whole process.
- Current system behaves as fail-fast.

### Realization

The Orchestrator is the natural place to introduce recovery/checkpointing because it coordinates the expensive TE and DE operations.

Future sessions will cover:

- persisted workflow state
- idempotency
- retry strategy
- DLQ
- async processing

---

## Reflection

Today's JWT session became temporarily overwhelming when OAuth concepts leaked into JWT.

Correction:

- Finish one abstraction at a time.
- JWT first.
- OAuth later.

This reinforced the importance of conceptual boundaries.
