# Circuit Breakers

A circuit breaker protects a caller and its downstream dependency from repeated calls when that dependency is persistently unhealthy.

## Why it exists

Retries help with transient failures, but repeated retries against a prolonged outage amplify load, consume threads/connections and can cause cascading failure.

## States

- **Closed**: requests flow normally; relevant downstream failures are counted.
- **Open**: calls fail fast or are deferred; the unhealthy dependency is not continuously hammered.
- **Half-Open**: a limited number of probe requests test whether the dependency has recovered. Success closes the circuit; failure opens it again.

## Failure classification

Typical retryable/health-relevant failures include timeouts, connection resets, `429`, `502`, `503` and `504`.

Client or business failures such as `400`, authentication/authorization failures and business validation failures should not be retried and should not increment the downstream circuit breaker.

## Retry and circuit breaker together

- Retry handles short transient failures.
- Circuit breaker handles persistent downstream failure.
- Exponential backoff and jitter reduce synchronized retry amplification.
- Long-running durable work should normally be rescheduled through a delayed queue retry rather than retried immediately inside the worker.

## Recovery surge

Closing a circuit must not release an entire backlog at once. Use controlled ramp-up, bounded concurrency and rate limiting so recovery does not create a thundering herd or trigger fresh `429` responses.

Adaptive rate limiting and adaptive concurrency algorithms were introduced but intentionally deferred for a dedicated lesson.

## Interview wording

> Retries absorb transient failures; a circuit breaker prevents persistent failures from cascading. I classify failures carefully, use backoff and jitter, probe recovery in half-open state, and release queued work gradually after recovery.
