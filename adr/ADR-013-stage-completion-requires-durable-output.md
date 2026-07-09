# ADR-013: Mark a Stage Complete Only After Durable Output Exists

- **Status:** Accepted
- **Date:** Day 10

## Context

A processing service may return a result before that result is durably stored. If the workflow marks a stage complete too early and then crashes, recovery may not have the artifact needed by the next stage.

Examples:

```text
TE returns OCR result
DE returns field predictions
PDF preparation generates page images
```

## Decision

A stage is marked complete only after its durable output exists and SQL points to it.

For every asynchronous stage:

```text
service returns result
→ durable output is stored
→ SQL stage state is updated with output references/status
→ outbox row is inserted for the next async action, if any
→ current queue message is acknowledged
```

## Consequences

- stage state represents recoverable progress, not merely an in-memory service response;
- redelivery can reconcile SQL and durable artifacts before repeating work;
- queue acknowledgement happens only after durable state is safe;
- exact durable-output representation may differ by stage.

## Stage Examples

```text
PDF preparation complete
→ page images/artifacts stored + SQL references exist

OCR complete
→ OCR artifact stored + SQL OCR reference exists

Field extraction complete
→ prediction output stored + SQL prediction-ready state exists
```

## Not Decided Here

- exact PDF image artifact format/path;
- exact field-extraction output persistence model;
- lifecycle/retention policy for intermediate artifacts.
