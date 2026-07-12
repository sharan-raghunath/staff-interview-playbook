# ADR-011: Keep Billing Outside the OCR and Field-Extraction Orchestrator

- **Status:** Accepted
- **Date:** Session 9

## Context

Orchestrator owns the machine-driven document-processing workflow:

```text
OCR → field extraction → predictions ready for review
```

Invoice Manager users may review and override predicted values. Those overrides are relevant to pricing. Therefore, pricing must use the final user-confirmed state rather than raw extraction output.

## Decision

Do not make Orchestrator trigger Billing immediately after field extraction.

Instead:

```text
Predictions ready
→ user review / override
→ user submits
→ IM REST API persists final submitted state
→ Billing is initiated independently
```

The internal IM REST API owns the business transition from review-ready data to submitted-for-billing data. Billing may be invoked synchronously or through a dedicated Billing work queue based on its latency/retry requirements; that implementation is not decided here.

## Consequences

- pricing uses final reviewed inputs;
- Orchestrator remains focused on OCR and field extraction;
- API owns the user-confirmed business transition;
- billing has an independent responsibility and future reliability boundary.
