# Invoice Manager Architecture Addendum (Day 7)

## Purpose
This document supplements the existing architecture documentation without exposing proprietary implementation details.

## Logical Flow
Client Web
 -> Akamai (WAF, DDoS, IP Whitelisting)
 -> Azure Application Gateway
 -> Traefik Ingress
 -> IM Web / Edge API
 -> IM REST API
 -> Orchestrator
 -> Text Extraction Service
 -> Data Extraction Service
 -> Source System

## Identity
Client IdP -> Federation -> FIS Identity Provider -> Microsoft Entra -> Invoice Manager

## Responsibilities

### IM Web / Edge API
- Hosts Angular SPA
- Authenticates users with Entra
- Creates server-side session
- BFF for browser

### IM REST API
- Business APIs
- Validates every incoming token
- Authorization
- Never blindly trusts Edge API

### Orchestrator
- Coordinates TE/DE
- Obtains service tokens
- No invoice ownership

### Text Extraction Service
- Abstracts OCR implementation
- May invoke Azure Computer Vision and/or Azure Document Intelligence
- Returns extracted text

### Data Extraction Service
- Performs ML / GenAI prediction
- Returns predicted invoice fields

## Infrastructure
- AKS hosts application services
- Traefik Ingress
- Azure SQL
- Azure Redis Cache (distributed cache)
- Azure Key Vault
- Azure App Configuration
- Application Insights + Log Analytics
- Availability Checks
- Jump Box for administration

## Redis
Redis is used as distributed cache to avoid instance affinity and share transient state across replicas.

## Data Ownership
Invoice Manager is NOT the System of Record for invoices.
It owns:
- Prediction overrides
- Pricing metadata
- Operational metadata
Future:
- Processing state
- Job lifecycle
- Idempotency metadata

Current system remains synchronous and fail-fast.
Future async architecture is documented separately.
