# Interview Mistakes and Corrections

## General

- Avoid jumping to technology before explaining the problem.
- Avoid memorized definitions without trade-offs.
- Clarify architecture assumptions before answering.

## Coding

- Trust a guaranteed-valid input contract; do not add defensive branches merely to appear production-ready in a constrained coding problem.
- For postfix evaluation, pop the right operand before the left operand.

- Do not ask permission for standard collections unless the interviewer has stated auxiliary-space or library restrictions; ask about constraints instead.
- Do not over-engineer for hypothetical delimiter types when the input contract is fixed.
- When `Peek` is immediately followed by `Pop`, check whether `TryPop` expresses the operation more directly.

- Do not ask for hints too early.
- Explain brute force before optimized solution.
- Narrate the optimized approach before coding.
- Watch for indexing bugs in sliding window problems.
- Prefer readable code over overly compact code.

## Authentication

- Do not confuse OAuth and JWT.
- Do not say OAuth is an alternative to JWT.
- Do not use ID Tokens for API authorization.
- Do not assume every microservice needs its own App Registration.
- Do not propagate user identity to services that do not need it.

## System Design

- Do not recommend caching merely because data exists; evaluate size, reuse, volatility, stale-data impact, security and the existing durable store.
- Do not treat invalidating shared L2 as sufficient when pod-local L1 copies can remain stale.

- Do not start with a product choice before explaining why asynchronous processing is needed.
- Do not conflate Correlation ID and Idempotency Key.
- Do not retry everything blindly.
- Do not drop failed work after retries without preserving it for recovery.


## Process consistency: sessions are not calendar days

A session may span multiple real-world days. Do not evaluate progress or refer to corrections using “today” or “yesterday.” Use the session number and the exact earlier point in that session.

Also avoid silently changing assumptions during a problem. State any assumption change before reasoning from it.
