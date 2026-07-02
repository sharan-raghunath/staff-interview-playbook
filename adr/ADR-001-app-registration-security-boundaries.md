# ADR-001: App Registrations Follow Security Boundaries

## Status

Accepted.

## Context

A question came up whether every microservice should have its own App Registration.

Invoice Manager contains several services:

- Edge API / Web App
- API Service
- Orchestrator
- Text Extractor
- Data Extractor

Orchestrator, TE and DE currently share an App Registration.

## Decision

App Registrations should align with **security boundaries**, not necessarily deployment boundaries.

## Rationale

If multiple services are tightly coupled, are not consumed independently, and share the same authorization requirements, they can reasonably share an App Registration.

If a component becomes an independently consumed platform capability, it should get its own identity.

## Consequences

- Reduces operational complexity where services form one protected resource.
- Maintains clear identity boundaries where independent authorization is required.
- Avoids dogmatic “one microservice = one App Registration” design.
