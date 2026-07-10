# Backpressure and Bounded Concurrency

## Core idea

A queue can absorb a burst, but it cannot increase downstream capacity.

When backlog grows, identify the constrained component before scaling.

## Investigation branches

- Consumer/orchestrator saturation: consider scaling consumers.
- TE saturation with downstream capacity available: consider scaling TE.
- External OCR provider returning `429`: reduce pressure, delay retries and use exponential backoff rather than scaling callers.
- Database or storage bottleneck: scaling application pods can worsen contention.

## Signals discussed

- queue depth;
- oldest-message age;
- arrival rate versus completion rate;
- requests in flight;
- stage latency;
- CPU and memory;
- pod restarts;
- internal and external `429` rates.

## Per-pod bounded concurrency

An asynchronous concurrency gate such as `.NET` `SemaphoreSlim` can cap in-flight operations inside one process or pod.

```text
Approximate total concurrency = per-pod limit × active pod count
```

This is not a strict global-concurrency solution. Global concurrency enforcement is intentionally deferred for a later session.

## Operational investigation order

> Metrics show what is happening. Traces show where it is happening. Logs help explain why.
