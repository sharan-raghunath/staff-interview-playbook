# ADR-005 - Platform and Application Responsibilities

Decision:
Separate application-owned services from Azure platform services.

Application-owned:
- IM Web
- IM REST API
- Orchestrator
- Billing
- Text Extraction
- Data Extraction

Platform services:
- Azure Computer Vision
- Azure Document Intelligence
- Azure SQL
- Azure Redis Cache
- Azure Key Vault
- Azure App Configuration
- Application Insights
- Log Analytics

Rationale:
Keeps service boundaries clear and simplifies interview explanations.
