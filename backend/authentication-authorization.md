# Authentication and Authorization

This document summarizes the completed authentication and authorization module.

## Mental Model

```text
HTTP
↓
Stateless
↓
Sessions
↓
Cookies
↓
Distributed Sessions
↓
JWT
↓
OAuth 2.0
↓
OIDC
```

Each step solves one problem and introduces new trade-offs.

---

## Authentication vs Authorization

Authentication answers:

> Who is the caller?

Authorization answers:

> What is the caller allowed to do?

A system can authenticate a user correctly and still deny the request during authorization.

Example:

- User is authenticated as Sharan.
- User tries to approve an invoice.
- API checks whether Sharan has the required role, scope, tenant access and resource permissions.

---

## Sessions and Cookies

HTTP is stateless. The server does not remember previous requests.

Sessions were introduced so applications could maintain user state such as:

- login state
- user identity
- roles and claims
- shopping cart
- tenant context

The browser stores only a Session ID, usually in a cookie. The server stores the actual session data.

---

## Distributed Sessions

In a multi-server environment, in-memory sessions fail because the next request may reach a different server.

A shared session store such as Redis solves this by becoming the single source of truth for sessions.

Trade-off:

- solves horizontal scaling
- introduces a network dependency in the request path
- Redis/database availability affects application availability

---

## JWT

JWT reduces the need for server-side session lookups by allowing the API to validate a signed token locally.

JWT improves scalability but makes immediate revocation harder.

See [`jwt.md`](jwt.md).

---

## OAuth 2.0

OAuth solves delegated authorization.

It allows an application to access a resource on behalf of a user without asking for the user's password.

See [`oauth.md`](oauth.md).

---

## OIDC

OIDC adds authentication on top of OAuth.

OAuth answers:

> Can this client access this resource?

OIDC answers:

> Who logged in?

See [`oidc.md`](oidc.md).

---

## RBAC vs ABAC

### RBAC

Role-Based Access Control checks roles.

Example:

```text
Role = Admin
```

RBAC is simple but can become too coarse for enterprise systems.

### ABAC

Attribute-Based Access Control evaluates attributes from:

- user
- resource
- action
- environment

Example:

```text
User.Tenant == Invoice.Tenant
AND User.Role == Reviewer
AND Invoice.Status == Pending
AND Invoice.Amount < 100000
```

ABAC is appropriate when authorization depends on business context.

---

## Roles vs Scopes

| Concept | Question Answered | Common Use |
|---------|-------------------|------------|
| Roles | Who are you in this application? | RBAC, app roles, service-to-service permissions |
| Scopes | What can this token do? | OAuth delegated permissions |

Roles represent business identity or application permissions.

Scopes represent delegated API permissions.

---

## User Identity vs Service Identity

User identity represents a human action.

Example:

```text
Sharan uploaded this invoice.
```

Service identity represents an application acting as itself.

Example:

```text
Orchestrator invoked Text Extractor.
```

A good system preserves the correct identity for the decision being made.

---

## Staff-Level Summary

> Authentication establishes who is calling. Authorization determines what they are allowed to do. Every protected resource should independently validate the caller according to its own security boundary.
