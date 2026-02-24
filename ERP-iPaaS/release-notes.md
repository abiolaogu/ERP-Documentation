# Release Notes -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## Release History

### v1.0.0 -- 2026-02-23 (Current Release)

**Release Type**: General Availability (GA)

#### Highlights

ERP-iPaaS v1.0.0 marks the General Availability release of the BillyRonks Integration Platform as a Service. This release delivers a production-ready integration backbone with six core services, 23 workflow templates, and comprehensive multi-tenant security.

#### New Features

**Workflow Engine**
- Activepieces 0.20.0 visual workflow builder with 100+ built-in actions
- Temporal 1.23.0 durable workflow execution with compensation/saga support
- Nexum Flow DAG engine with AI-augmented node types (HTTP, LLM, MCP, Temporal)
- 16 Activepieces workflow templates covering CRM, Finance, DevOps, Marketing, Data, and Support
- 7 Temporal durable workflow templates covering Lead Intake, Support Escalation, KYC Approval, On-Call Escalation, Rate Limit Shield, Invoice Reconciliation, and Usage Aggregation
- Human-in-the-loop approval workflows via Temporal signals

**Connector Framework**
- TypeScript SDK for custom connector development (`packages/integration-layer-ts`)
- Go SDK for server-side integrations (`packages/integration-layer-go`)
- Connector CLI for scaffolding, validation, and publishing (`packages/connector-cli`)
- 3 built-in Activepieces pieces: ClickHouse, ERPNext, Internal CRM
- Connector marketplace with quality scoring, ratings, and badges
- Auto-generated connectors from OpenAPI specifications

**Event Backbone**
- Redpanda (Kafka-compatible) event streaming
- Avro schema registry with backward-compatible evolution
- Tenant-scoped topics with dead-letter queues
- CloudEvents v1.0 envelope format
- WorkflowCommand Avro schema for cross-service events

**API Management**
- Traefik v2.10 API gateway with TLS termination
- Rate limiting per tenant and API key
- OAuth2 token validation via Keycloak
- Open Integration Layer API with 12 endpoints
- Developer portal scaffold

**ETL Service**
- Batch ETL pipeline management via Go service
- Extract from databases, APIs, and files
- Transform with mapping, filtering, and deduplication
- Load to ClickHouse, PostgreSQL, and MinIO

**Webhook Management**
- Incoming webhook registration with unique URLs
- HMAC-SHA256 signature verification
- Platform-specific signing (Zapier, Make, Power Automate, Pabbly, IFTTT, Integrately)
- Delivery logging and retry with exponential backoff

**Infrastructure**
- 16 Helm charts for Kubernetes deployment
- ArgoCD GitOps with app-of-apps pattern
- Terraform modules for cloud provisioning
- Kustomize overlays for dev/prod environments
- KEDA auto-scaling for workflow workers
- OPA Gatekeeper for policy enforcement

**Observability**
- Grafana 10.1 dashboards (WaaS Overview)
- Prometheus alerting (5 rules: worker saturation, task backlog, Kafka lag, API errors, cost spikes)
- Loki log aggregation
- Tempo distributed tracing
- Sentry error tracking

**Data**
- PostgreSQL 16 with Row-Level Security for tenant isolation
- ClickHouse 23.9 with 9 tables and 1 materialized view
- Dragonfly (Redis-compatible) caching
- MinIO S3-compatible object storage

**Security**
- Zero-trust multi-tenant architecture
- Keycloak 22.0 OAuth2/OIDC identity provider
- PostgreSQL RLS for database-level tenant isolation
- PII redaction in LLM utilities
- Encrypted secret storage with rotation policies
- Immutable audit trail in ClickHouse

**Interoperability**
- 6 platform interop mapping templates (Zapier, Make, Power Automate, Pabbly, IFTTT, Integrately)
- 2 interop recipes (Zapier-to-Activepieces, Activepieces-to-Power Automate)

#### Known Limitations

- Protobuf schema support in schema registry not yet enforced (Avro only)
- CDC via Debezium referenced but not deployed
- Python SDK for custom connectors not yet published
- DLQ replay requires CLI; no UI available
- ETL streaming mode not yet implemented (batch only)
- Visual step-through debugger not yet available

#### Migration Notes

This is the initial GA release. No migration steps required.

---

### Pre-Release History

| Version | Date | Notes |
|---------|------|-------|
| v0.9.0-rc1 | 2026-02-15 | Release candidate with all core services |
| v0.8.0-beta | 2026-01-15 | Beta with Temporal integration |
| v0.7.0-alpha | 2025-12-02 | Alpha with Activepieces + Redpanda |

## Upcoming Releases

### v1.1.0 (Planned: Q2 2026)

- Python SDK for custom connectors
- Debezium CDC deployment
- DLQ replay UI
- Protobuf schema enforcement
- Enhanced developer portal with API explorer

### v1.2.0 (Planned: Q3 2026)

- Visual step-through workflow debugger
- Streaming ETL mode
- GraphQL gateway
- Connector marketplace review workflow
- Visual ETL pipeline builder
