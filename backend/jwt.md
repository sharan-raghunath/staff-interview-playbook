# JWT

## Why JWT Exists

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

## Why the Browser Cannot Safely Modify a JWT

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

---

## Access Token vs ID Token

| Token | Audience | Purpose |
|-------|----------|---------|
| Access Token | API | Authorize API calls |
| ID Token | Client application | Tell the client who logged in |

APIs should not use ID Tokens for authorization.

---

## Refresh Tokens

Refresh Tokens allow the application to obtain a new Access Token without asking the user to log in again.

They exist because Access Tokens should be short-lived.

Refresh Tokens are sensitive and should be protected carefully.

---

## Refresh Token Rotation

Refresh Token Rotation reduces the risk of long-lived refresh tokens.

```text
RT1 used
↓
RT2 issued
↓
RT1 invalidated
```

If RT1 is reused later, it may indicate theft.

---

## Revocation

Pure JWT validation is stateless.

This makes immediate revocation difficult.

Options:

- short-lived Access Tokens
- Refresh Token revocation
- revocation list keyed by `jti`
- token introspection
- reference tokens

Trade-off:

A revocation list reintroduces a distributed lookup, which JWT was partly designed to avoid.

---

## When Not to Use Pure JWT

Pure stateless JWT may not be ideal when immediate revocation is mandatory.

Examples:

- banking
- trading
- healthcare
- government systems
- highly sensitive admin systems

In those systems, server-side sessions, reference tokens or introspection may be preferable.

---

## Staff-Level Summary

> JWT and server-side sessions optimize for different trade-offs. JWT improves scalability by enabling local token validation, but immediate revocation becomes harder. Sessions make revocation easier but require shared state in distributed deployments.
