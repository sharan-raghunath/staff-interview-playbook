# Identity Principles

## App Registration Represents a Security Boundary

An App Registration should represent a protected resource or security boundary, not blindly map to every microservice.

Example:

```text
Edge API / Web App       → App Registration A
API Service              → App Registration B
Invoice Processing Engine → App Registration C
```

Where Invoice Processing Engine may include:

- Orchestrator
- Text Extractor
- Data Extractor

if they are not independently consumed resources.

---

## User Identity vs Service Identity

### User Identity

Represents a human business actor.

Used for:

- business authorization
- audit
- ownership
- user-specific decisions

Example:

```text
Sharan submitted the invoice prediction to the source system.
```

### Service Identity

Represents an application or workload.

Used for:

- service-to-service authentication
- Zero Trust
- endpoint protection
- least privilege between services

Example:

```text
Orchestrator invoked Text Extractor.
```

---

## TE and DE Identity Model

Text Extractor and Data Extractor do not need end-user identity for business logic.

They should validate:

- token signature
- issuer
- audience
- expiry
- service role/scope such as `OCR.Invoke` or `Extraction.Invoke`

They should not need to know who Sharan is unless a future requirement makes user-specific processing necessary.

---

## OBO vs Client Credentials

Use On-Behalf-Of when downstream services need the user's identity.

Use Client Credentials when the service acts as itself.

Decision question:

> Whose action is represented by this operation?

If the answer is a user, use user identity/OBO.

If the answer is a workload, use service identity/client credentials.
