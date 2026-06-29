# Invoice Manager

## Architecture

Akamai -> Gateway -> Traefik -> Edge API -> API Service -> Orchestrator -> Text Extractor -> Data Extractor

## Design Decisions

### API Service vs Orchestrator
- API Service owns application-specific business logic.
- Orchestrator remains application-agnostic.

### Text Extractor vs Data Extractor
- OCR preprocessing separated from ML extraction.
- Independent scaling.
- Fault isolation.
