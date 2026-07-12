# Session 6

## Backend / Security

### Topics Covered

- OAuth 2.0 from first principles
- Why OAuth exists
- Password sharing problem before OAuth
- Delegated authorization
- User consent
- Client registration
- Redirect URI validation
- Authorization Code Flow
- Implicit Flow and why it is being replaced
- PKCE concept and public clients
- Client Credentials Flow
- On-Behalf-Of Flow concept
- OIDC
- ID Token vs Access Token
- Token lifecycle
- Refresh Tokens
- Logout
- SSO and SLO
- Authentication and authorization best practices

### Key Insights

OAuth is not an alternative to JWT.

- JWT is a token format.
- OAuth is a protocol for delegated authorization.
- OIDC adds authentication on top of OAuth.

The most important architecture principle derived today:

> Identity should follow security boundaries, not deployment units.

### Invoice Manager Authentication Clarifications

- Edge API and Web App are one public-facing service boundary.
- Invoice Manager currently uses Implicit Flow and is being migrated to Authorization Code Flow.
- Since the application is server-backed, PKCE is not necessarily the main migration target.
- Orchestrator, TE and DE can share an App Registration if they represent one internal processing security boundary.
- TE and DE should validate service identity, not end-user identity.

---

## Coding

### Problem

Maximum Average Subarray I.

### Pattern

Fixed-size Sliding Window.

### Key Insight

Since `k` is constant, maximizing average is equivalent to maximizing sum.

Maintain current window sum and max sum.

When the window slides:

```text
sum = sum - outgoing + incoming
```

### Bug Fixed

Initial bug used:

```csharp
nums[k - i]
```

Correct outgoing index:

```csharp
nums[i - k]
```

### Complexity

- Time: O(n)
- Space: O(1)

---

## System Design

### Topic

Invoice Manager async/resilience motivation.

### Covered

- Long-running processing should not be coupled to HTTP request lifetime.
- Async API should return `202 Accepted` with `jobId` and `statusUrl`.
- Polling is simple but can create load at scale.
- Idempotency prevents duplicate processing when clients retry.
- Correlation ID and Idempotency Key are different concepts.
- Retry strategy should distinguish transient vs permanent failures.
- DLQ preserves failed work for investigation and recovery.

### Key Staff-Level Phrase

> The business operation can outlive the HTTP request, so the workflow should be decoupled from the request lifecycle.

---

## Reflection

Session 6 completed the Authentication and Authorization module for Staff interview purposes.

The discussion moved from protocol mechanics into architecture principles:

- security boundaries
- service identity
- user identity
- Zero Trust
- token purpose
- operational recovery
- idempotency

Next major focus: full Invoice Manager async redesign.
