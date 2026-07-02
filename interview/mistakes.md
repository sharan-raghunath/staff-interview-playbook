# Interview Mistakes and Corrections

## General

- Avoid jumping to technology before explaining the problem.
- Avoid memorized definitions without trade-offs.
- Clarify architecture assumptions before answering.

## Coding

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

- Do not start with Kafka/queues before explaining why async is needed.
- Do not conflate Correlation ID and Idempotency Key.
- Do not retry everything blindly.
- Do not drop failed work after retries without preserving it for recovery.
