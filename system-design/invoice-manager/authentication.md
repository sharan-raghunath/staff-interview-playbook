# Invoice Manager Authentication and Identity Architecture

## Canonical Boundary

Invoice Manager's public boundary is:

```text
Browser
  ↓
Edge API / Web App
```

The Angular Web App and Edge API backend should be treated as one public-facing service boundary for architecture discussions.

---

## Current / Target Login Direction

Invoice Manager currently uses Implicit Flow and is being migrated to Authorization Code Flow.

Because the application is server-backed, the primary improvement is moving away from tokens returned directly through the browser.

PKCE is primarily critical for public clients such as pure SPAs and mobile apps. It may still be supported depending on configuration, but the key target is Authorization Code Flow with server-side client authentication.

---

## Identity Provider

Microsoft Entra acts as the Identity Provider / Authorization Server.

It owns:

- user authentication
- MFA
- Conditional Access
- token issuance
- signing keys
- discovery metadata
- JWKS

---

## Downstream Calls

```text
Edge API / Web App
  ↓
API Service
  ↓
Orchestrator
  ├── Text Extractor
  └── Data Extractor
```

Identity design depends on whose action is represented.

### User-Aware Operations

If API Service needs to authorize based on the user, preserve user identity using an appropriate delegated/OBO-style model.

Examples:

- user submits prediction result
- user-specific authorization
- tenant-aware business decision
- audit requiring the user

### Service Operations

If a service acts as itself, use service identity / Client Credentials.

Examples:

- Orchestrator invokes TE
- Orchestrator invokes DE
- service reads Key Vault
- health monitoring
- scheduled cleanup

---

## TE / DE Identity

TE and DE should authenticate and authorize the calling service, but do not need the end-user identity for normal processing.

They can validate service roles/scopes such as:

```text
OCR.Invoke
Extraction.Invoke
InvoiceProcessing.Internal
```

This preserves Zero Trust without coupling TE/DE to user claims.

---

## App Registration Principle

Use App Registrations per security boundary.

Potential model:

```text
Edge API / Web App        → App Registration A
API Service               → App Registration B
Invoice Processing Engine → App Registration C
```

Invoice Processing Engine may include:

- Orchestrator
- Text Extractor
- Data Extractor

if they collectively form one internal protected resource.

---

## Staff-Level Talking Point

> From the transport perspective, Edge API calls API Service. From the business perspective, some operations are performed on behalf of the user. A well-designed system preserves both concepts: service identity for secure service-to-service calls, and user identity where authorization or audit requires it.
