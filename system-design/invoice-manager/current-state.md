# Invoice Manager Current State

## Flow

```text
User uploads document
↓
Malware scan
↓
Edge API
↓
API Service
↓
Orchestrator
↓
Text Extractor
↓
Data Extractor
↓
Response returned to user
```

---

## Current Behaviour

The current architecture is largely synchronous.

The user waits while:

- document is processed
- OCR is extracted
- ML / GenAI extraction runs
- response is aggregated

If a downstream component fails or times out, the request fails.

---

## Known Bottlenecks

Text Extractor and Data Extractor are the most time-intensive components.

Data Extractor can become the bottleneck because:

- models are loaded per field
- custom NER models execute sequentially or in limited parallelism
- GenAI extraction may add latency
- large invoices increase processing time

---

## Timeout Issue

If processing takes longer than the configured timeout, the user receives an error.

Previously discussed example:

- Akamai timeout around 4 minutes
- Internal service-to-service timeout around 10 minutes

TODO:

- Confirm exact timeout values for each layer.

---

## Failure Behaviour

If Orchestrator crashes mid-request:

- the request fails
- user receives a 500-style error
- progress is not recoverable in the current synchronous flow
- retry may restart work from the beginning

The system currently behaves as a fail-fast design without durable recovery.

---

## Recovery Gap

The key gap is that orchestration progress is not durably checkpointed.

Example failure:

```text
OCR completed
↓
Orchestrator crashes before Data Extractor starts
↓
OCR result may be lost from workflow perspective
```

Future design should consider:

- persisted workflow state
- idempotency keys
- stage-level checkpoints
- retryable commands
- async processing
