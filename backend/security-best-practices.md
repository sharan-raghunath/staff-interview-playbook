# Authentication and Authorization Best Practices

## Token Validation

Every protected API should validate:

- signature
- issuer (`iss`)
- audience (`aud`)
- expiry (`exp`)
- not-before (`nbf`) if present
- required scopes/roles
- tenant rules where applicable

Do not validate only the signature.

---

## Token Purpose

Never use a token outside its intended audience and purpose.

- ID Token: for client authentication context
- Access Token: for API authorization

APIs should reject ID Tokens.

---

## Short-Lived Access Tokens

Access Tokens are bearer credentials.

If stolen, whoever holds them can use them until expiry.

Keep them short-lived to reduce blast radius.

---

## Refresh Token Protection

Refresh Tokens are sensitive because they can obtain new Access Tokens.

Use:

- secure storage
- rotation
- reuse detection
- revocation
- sensible lifetimes

---

## Least Privilege

Grant only the permissions needed.

Prefer:

```text
invoice.read
```

instead of:

```text
invoice.*
```

where possible.

---

## Zero Trust

Do not trust:

- browsers
- internal networks
- upstream services by default

Trust:

- cryptographically validated tokens
- trusted Identity Providers
- explicitly configured authorization policies

---

## Identity Boundaries

Use one App Registration per independent security boundary.

Not necessarily one per microservice.

A security boundary may contain multiple tightly coupled internal services.

---

## Identity Propagation

Propagate end-user identity only where required for:

- authorization
- auditing
- business rules

Do not propagate user claims to services that do not need them.

---

## Operational Context

Propagate operational context separately from identity:

- correlation ID
- trace ID
- request ID
- job ID
- tenant ID where needed

---

## Staff-Level Principle

> Authentication establishes who is calling. Authorization determines what they can do. Every protected resource should independently validate both according to its own security boundary.
