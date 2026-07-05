# Invoice Manager Current State

## Flow

```text
User uploads document
↓
Malware scan
↓
Edge API / Web App
↓
API Service
↓
Orchestrator
↓
Text Extractor
↓
Data Extractor
↓
Predictions returned to user
↓
User reviews/overrides predictions
↓
Submit final result to source system
```

---

## Current Behaviour

The current architecture is largely synchronous during prediction.

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

If processing takes longer than a gateway/browser/request timeout, the user receives an error even if downstream processing later completes.

Possible outcomes:

- 499 Client Closed Request if the client disconnects
- 504 Gateway Timeout if a gateway times out
- 500-style application error depending on where failure is handled

The expensive processing result may be wasted because no caller remains to receive it.

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

Target-state direction already discussed:

- durable job/workflow state;
- idempotency keys;
- asynchronous processing;
- recovery from a persisted completed stage rather than assuming the original HTTP request remains alive.
