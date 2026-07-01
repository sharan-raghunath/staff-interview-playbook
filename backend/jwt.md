# JWT

## Why JWT exists

JWT was introduced as one way to reduce dependency on server-side session storage.

Evolution:

```text
HTTP is stateless
↓
Applications need user state
↓
Sessions store user state on the server
↓
Cookies carry Session ID
↓
Multiple servers break in-memory sessions
↓
Redis/shared store solves distributed sessions
↓
Redis becomes a dependency in every request
↓
JWT allows local token validation
```

JWT is not automatically better than sessions.

It optimizes for stateless validation and horizontal scalability, but immediate revocation becomes harder.

---

## JWT Structure

A JWT has three parts:

```text
header.payload.signature
```

### Header

Contains metadata about the token.

Example:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "A123XYZ"
}
```

Important fields:

- `alg` — signing algorithm
- `typ` — token type
- `kid` — key identifier

### Payload

Contains claims.

Claims are facts about the token, user, client or authorization context.

### Signature

The signature proves that the header and payload were issued by a trusted authority and were not modified.

---

## Signature Validation

In enterprise systems, JWTs are commonly signed using asymmetric cryptography.

```text
Identity Provider
  ↓
Signs token using private key

API
  ↓
Validates token using public key
```

The private key stays with the Identity Provider.

The API receives only the public key.

This allows APIs to validate tokens without contacting the Identity Provider on every request.

---

## Why the browser cannot safely modify a JWT

The browser/user can technically edit the token text.

However, if the payload changes, the signature no longer matches.

Example:

```json
{
  "role": "User"
}
```

If someone changes it to:

```json
{
  "role": "Admin"
}
```

the API will detect tampering during signature validation.

Staff phrasing:

> JWT does not prevent the client from modifying token contents. It makes tampering detectable because the signature is computed over the header and payload.

---

## `kid` and JWKS

Identity providers rotate signing keys.

During rotation, multiple public keys may be valid at the same time.

The JWT header contains:

```json
{
  "kid": "A123XYZ"
}
```

`kid` means **Key Identifier**.

It tells the API which public key to use.

---

## JWKS Flow

1. API receives JWT.
2. API reads header.
3. API extracts `kid`.
4. API checks cached JWKS.
5. API finds matching public key.
6. API validates signature.

JWKS = JSON Web Key Set.

It is a list of public keys published by the Identity Provider.

---

## Key Rotation

Identity providers rotate keys for security.

Normal flow:

```text
Token signed with Key A
↓
API validates with cached public Key A
```

After rotation:

```text
New token signed with Key B
↓
API sees unknown kid
↓
API refreshes JWKS
↓
API finds public Key B
↓
Validation succeeds
```

Old keys remain published until tokens signed with them expire.

This prevents valid existing tokens from breaking immediately after key rotation.

---

## Important Claims

| Claim | Meaning |
|------|---------|
| `iss` | Issuer — who created the token |
| `aud` | Audience — who the token is intended for |
| `sub` | Subject — principal represented by token |
| `oid` | Object ID, commonly Azure Entra user identifier |
| `tid` | Tenant ID |
| `exp` | Expiry |
| `nbf` | Not valid before |
| `iat` | Issued at |
| `scp` | Delegated scopes |
| `roles` | App roles / application permissions |
| `jti` | JWT ID, useful for revocation lists |

---

## Validation Checklist

APIs commonly validate:

- signature
- issuer
- audience
- expiry
- not-before
- required scopes or roles
- tenant rules if multi-tenant

If authentication validation fails, return `401 Unauthorized`.

If authentication succeeds but authorization fails, return `403 Forbidden`.

---

## ID Token vs Access Token

### ID Token

Purpose:

> Tell the client application who the user is.

Consumer:

> Client application.

Used for:

- login
- user identity
- profile information
- local session creation

### Access Token

Purpose:

> Allow an API to authorize a request.

Consumer:

> API.

Used for:

- API access
- scopes
- roles
- audience validation

Important rule:

> APIs should validate Access Tokens, not ID Tokens.

An ID Token may contain user identity, but its audience is typically the client application, not the API.

---

## Roles vs Scopes

### Roles

Roles answer:

> Who are you in this application?

Examples:

- Admin
- Reviewer
- Approver

Roles are commonly used for RBAC.

### Scopes

Scopes answer:

> What can this token do?

Examples:

- `invoice.read`
- `invoice.write`
- `invoice.approve`

Scopes are common in delegated OAuth flows.

---

## RBAC vs ABAC

### RBAC

Role-Based Access Control.

Example:

```text
Role == Admin
↓
Allow
```

Good for simple permissions.

### ABAC

Attribute-Based Access Control.

Evaluates attributes from:

- user
- resource
- action
- environment

Example:

```text
User.Tenant == Invoice.Tenant
AND Invoice.Status == Pending
AND User.Role == Reviewer
```

Invoice Manager is a strong ABAC-style system because authorization depends on tenant, invoice state, user attributes and business rules.

---

## Refresh Tokens

Access Tokens are usually short-lived.

Why not issue long-lived Access Tokens?

Because if stolen, attackers can call APIs until expiry.

Refresh Tokens exist so users do not need to log in again every time an Access Token expires.

Flow:

```text
Access Token expires
↓
Client sends Refresh Token to IdP
↓
IdP issues new Access Token
```

Refresh Tokens are sent to the Identity Provider, not to APIs.

---

## Refresh Token Rotation

Without rotation:

```text
Same Refresh Token works repeatedly
```

If stolen, attacker can keep getting new Access Tokens.

With rotation:

```text
R1 used
↓
IdP issues R2
↓
R1 invalidated
```

If R1 is reused, the IdP can detect possible compromise and revoke the token chain.

---

## JWT Revocation

Pure JWT validation is stateless.

That is both its strength and weakness.

If a user logs out, an unexpired Access Token may still validate successfully because the API does not call the IdP on every request.

Options:

### Short-lived Access Tokens

Most common.

Limits exposure window.

### Revocation List / Blacklist

Store revoked token IDs, often by `jti`, in Redis or a database.

Every request checks whether the token was revoked.

This supports immediate logout but reintroduces a distributed lookup.

### Token Introspection

API asks the IdP whether the token is still active.

Stronger revocation semantics, but adds latency and dependency.

### Reference Tokens

Token is an opaque reference.

API looks up token details from the authorization server.

More session-like.

---

## JWT Trade-offs

JWT is good when:

- APIs need scalable token validation
- microservices need local validation
- short revocation delay is acceptable
- distributed session storage is undesirable

JWT may not be ideal when:

- immediate logout is mandatory
- user disablement must take effect instantly
- banking / trading / highly regulated systems require real-time revocation
- token size becomes large
- claims change frequently

---

## Staff Interview Answer

Question:

> Is JWT better than sessions?

Strong answer:

> JWT and server-side sessions optimize for different trade-offs. JWT improves scalability by allowing APIs to validate tokens locally without a shared session store, but immediate revocation becomes harder. Server-side sessions make revocation straightforward but require centralized session storage for distributed deployments. The right choice depends on the application's security, scalability and operational requirements.

---

## Common Mistakes

- Treating ID Tokens as API Access Tokens.
- Saying JWT is always stateless and therefore always better.
- Ignoring revocation.
- Putting sensitive data in JWT payload.
- Not validating audience.
- Not validating issuer.
- Confusing roles and scopes.
- Assuming signature validation alone is sufficient.
