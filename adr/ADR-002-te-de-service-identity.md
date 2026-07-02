# ADR-002: Text Extractor and Data Extractor Use Service Identity

## Status

Accepted.

## Context

TE and DE are internal processing services.

TE performs document preprocessing and OCR.

DE performs ML/GenAI extraction.

They do not make business decisions based on the end user.

## Decision

TE and DE should authenticate and authorize the calling service using service identity.

They should not require end-user identity for normal processing.

## Rationale

TE and DE need to know that the caller is an authorized internal service, not which end user triggered the workflow.

This preserves Zero Trust while avoiding unnecessary coupling to user claims.

## Consequences

- Simpler TE/DE contracts.
- Clearer service responsibility boundaries.
- Reduced token/claims propagation.
- Still enforces endpoint-level security using service roles/scopes.
