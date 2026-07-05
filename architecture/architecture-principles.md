# Architecture Principles

This file captures principles derived during Interview Prep.

## 1. Responsibilities Before Technologies

Start with:

> What responsibility does this component own?

Do not start with:

> Which technology should we use?

Technology choices should follow responsibility boundaries and business requirements.

---

## 2. Security Boundary Before Deployment Boundary

A microservice is a deployment unit.

An App Registration or identity boundary should represent a protected resource or security boundary.

Not every microservice automatically needs its own App Registration.

---

## 3. One App Registration per Security Boundary

Use separate App Registrations when components are independently protected resources.

Tightly coupled internal services can share an App Registration when they collectively form one logical protected resource and have the same authorization requirements.

---

## 4. Propagate User Identity Only When Needed

Do not blindly pass user tokens through every service.

Propagate user identity when needed for:

- authorization
- audit
- business decisions

Use service identity when the downstream service only needs to trust the calling application.

---

## 5. Operational Context Is Not Identity

Correlation IDs, trace IDs, request IDs, tenant IDs and job IDs are operational context.

They are not substitutes for user or service identity.

---

## 6. Every Identifier Should Have One Responsibility

| Identifier | Responsibility |
|------------|----------------|
| Request ID | One HTTP request |
| Correlation ID | Distributed tracing |
| Idempotency Key | Business deduplication |
| Job ID | Async workflow tracking |

Do not reuse one identifier for multiple responsibilities.

---

## 7. Long-Running Work Should Not Be Coupled to HTTP Lifetimes

HTTP request lifetimes are not suitable for long-running business operations.

If business processing can outlive gateway/browser/request timeouts, return quickly and process asynchronously.

---

## 8. Retry Requires Idempotency

Retries are dangerous unless the operation can be safely repeated.

Design idempotency before adding retries.

---

## 9. Preserve Failed Work for Recovery

If retries are exhausted, preserve the message/job in a Dead Letter Queue or equivalent recovery mechanism.

Do not simply lose the work or force users to re-upload without investigation.

---

## 10. Staff-Level Design Question

For every design decision, ask:

> What problem are we solving, what trade-off are we accepting, and how will this operate in production?
