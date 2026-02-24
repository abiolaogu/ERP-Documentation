# Workflows and User Journeys -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Overview

ERP-iPaaS provides 23+ pre-built workflow templates across two runtime engines (Activepieces and Temporal), covering lead management, support operations, finance, HR, commerce, data engineering, and DevOps workflows.

## 2. Workflow Template Inventory

### 2.1 Activepieces Templates (16)

| # | Template | File | Category |
|---|---------|------|----------|
| 1 | Lead Intake | `lead-intake.json` | CRM |
| 2 | PR Quality Gate | `pr-quality-gate.json` | DevOps |
| 3 | Sheet-to-DB Sync | `sheet-db-sync.json` | Data |
| 4 | Marketing Syndication | `marketing-syndication.json` | Marketing |
| 5 | Support Triage | `support-triage.json` | Support |
| 6 | Commerce Order Ops | `commerce-order-ops.json` | Commerce |
| 7 | Social Listening | `social-listening.json` | Marketing |
| 8 | Finance ETL | `finance-etl.json` | Finance |
| 9 | Calendar Brief | `calendar-brief.json` | Productivity |
| 10 | Data Lake Ingest | `data-lake-ingest.json` | Data |
| 11 | Alerting On-Call | `alerting-oncall.json` | DevOps |
| 12 | Invoice to ERP | `invoice-to-erp.json` | Finance |
| 13 | HR Onboarding | `hr-onboarding.json` | HCM |
| 14 | KYC Intake | `kyc-intake.json` | Compliance |
| 15 | Rate Limit Shield | `rate-limit-shield.json` | Platform |
| 16 | Batch to Streaming | `batch-to-streaming.json` | Data |

### 2.2 Temporal Templates (7)

| # | Template | File | Category |
|---|---------|------|----------|
| 17 | Lead Intake (durable) | `lead-intake.json` | CRM |
| 18 | Support Escalation | `support-escalation.json` | Support |
| 19 | KYC Approval | `kyc-approval.json` | Compliance |
| 20 | On-Call Escalation | `oncall-escalation.json` | DevOps |
| 21 | Rate Limit Shield | `rate-limit-shield.json` | Platform |
| 22 | Invoice Reconciliation | `invoice-reconciliation.json` | Finance |
| 23 | Usage Aggregation | `usage-aggregation.json` | Billing |

## 3. Core User Journeys

### 3.1 Journey: Business Analyst Creates a Workflow

```mermaid
sequenceDiagram
    participant BA as Business Analyst
    participant UI as Activepieces UI
    participant AP as Activepieces Runtime
    participant RP as Redpanda
    participant CH as ClickHouse

    BA->>UI: Open visual builder
    BA->>UI: Browse template marketplace
    BA->>UI: Select "Lead Intake" template
    UI->>AP: Import template
    BA->>UI: Configure trigger (webhook)
    BA->>UI: Map fields to CRM schema
    BA->>UI: Add Slack notification action
    BA->>UI: Test with sample payload
    UI->>AP: Execute test run
    AP->>RP: Publish test event
    RP->>CH: Log run to analytics
    AP-->>UI: Return test results
    BA->>UI: Activate workflow
    UI->>AP: Enable trigger
    AP-->>BA: Workflow live
```

### 3.2 Journey: Developer Builds a Custom Connector

```mermaid
sequenceDiagram
    participant DEV as Developer
    participant CLI as Connector CLI
    participant REG as Schema Registry
    participant MKT as Marketplace
    participant VAL as Validation Pipeline

    DEV->>CLI: npx connector-cli scaffold my-connector
    CLI-->>DEV: Generated project structure
    DEV->>DEV: Implement auth + actions
    DEV->>CLI: npx connector-cli validate
    CLI->>REG: Validate schemas
    REG-->>CLI: Schema valid
    CLI-->>DEV: Validation passed
    DEV->>CLI: npx connector-cli publish
    CLI->>MKT: Upload connector bundle
    MKT->>VAL: Trigger validation job
    VAL-->>MKT: Quality score: 87/100
    MKT-->>DEV: Published v1.0.0
```

### 3.3 Journey: Platform Admin Monitors Integrations

```mermaid
sequenceDiagram
    participant ADMIN as Platform Admin
    participant GF as Grafana
    participant PM as Prometheus
    participant CH as ClickHouse
    participant PD as PagerDuty

    ADMIN->>GF: Open WaaS Overview Dashboard
    GF->>PM: Query workflow metrics
    GF->>CH: Query run analytics
    GF-->>ADMIN: Display dashboard

    Note over PM: Alert: TemporalTaskBacklog > 1000

    PM->>PD: Fire critical alert
    PD-->>ADMIN: Page notification
    ADMIN->>GF: Drill into Temporal metrics
    ADMIN->>GF: Check worker saturation
    ADMIN->>ADMIN: Scale workers via KEDA
    PM-->>ADMIN: Alert resolved
```

### 3.4 Journey: Data Engineer Builds ETL Pipeline

```mermaid
sequenceDiagram
    participant DE as Data Engineer
    participant API as ETL Service API
    participant ETL as ETL Runtime
    participant SRC as Source DB
    participant TGT as ClickHouse
    participant RP as Redpanda

    DE->>API: POST /v1/etl (pipeline definition)
    API-->>DE: Pipeline created
    DE->>API: POST /v1/etl/{id}/run
    API->>ETL: Trigger pipeline execution
    ETL->>SRC: Extract (SELECT with pagination)
    ETL->>ETL: Transform (map, filter, deduplicate)
    ETL->>TGT: Load (batch INSERT)
    ETL->>RP: Publish etl.pipeline.completed event
    RP->>TGT: Log metrics
    ETL-->>DE: Pipeline run completed
```

## 4. Workflow Architecture Patterns

### 4.1 Simple Sequential Workflow

```mermaid
graph LR
    T[Trigger<br/>Webhook] --> A1[Action 1<br/>Fetch Data]
    A1 --> A2[Action 2<br/>Transform]
    A2 --> A3[Action 3<br/>Send Email]
```

### 4.2 Conditional Branching Workflow

```mermaid
graph TD
    T[Trigger] --> C{Condition<br/>deal_value > 10000?}
    C -->|Yes| A1[Enterprise Sales<br/>Assign VP]
    C -->|No| A2[Standard Sales<br/>Assign Rep]
    A1 --> N[Notify CRM]
    A2 --> N
```

### 4.3 Parallel Fan-Out Workflow

```mermaid
graph TD
    T[Trigger<br/>New Order] --> P{Parallel<br/>Fan-Out}
    P --> A1[Update Inventory]
    P --> A2[Generate Invoice]
    P --> A3[Notify Warehouse]
    A1 --> J{Join}
    A2 --> J
    A3 --> J
    J --> F[Send Confirmation]
```

### 4.4 Durable Saga Workflow (Temporal)

```mermaid
graph TD
    T[Start Saga] --> A1[Reserve Inventory]
    A1 --> A2[Charge Payment]
    A2 --> A3[Create Shipment]
    A3 --> S[Saga Complete]

    A2 -->|Payment Failed| C1[Compensate: Release Inventory]
    A3 -->|Shipment Failed| C2[Compensate: Refund Payment]
    C2 --> C1
    C1 --> F[Saga Failed]
```

### 4.5 Human-in-the-Loop Workflow

```mermaid
graph TD
    T[Trigger<br/>KYC Document Submitted] --> A1[Extract Document Data]
    A1 --> A2[LLM Risk Assessment]
    A2 --> C{Risk Score<br/>> 0.7?}
    C -->|High Risk| H[Human Approval<br/>Signal Wait]
    C -->|Low Risk| A3[Auto Approve]
    H -->|Approved| A3
    H -->|Rejected| R[Reject Application]
    A3 --> N[Notify Applicant]
    R --> N
```

## 5. Workflow Template Details

### 5.1 Lead Intake Workflow (Temporal)

```typescript
export async function leadIntakeWorkflow(input: LeadIntakeInput) {
  // Step 1: Upsert lead in CRM
  await upsertLeadActivity({
    tenantId: input.tenantId,
    crmBaseUrl: process.env.CRM_BASE_URL,
    token: process.env.CRM_TOKEN,
    payload: input.lead,
  });

  // Step 2: Generate summary via LLM
  const summary = await llmActivity({
    tenantId: input.tenantId,
    type: 'marketing-copy',
    instructions: 'Generate a friendly summary for sales team.',
    context: input.lead,
  });

  // Step 3: Notify sales channel
  await notifyChannel({
    tenantId: input.tenantId,
    channel: 'slack',
    endpoint: input.slackWebhook,
    message: `New lead captured: ${summary.headline}`,
    metadata: summary,
  });
}
```

### 5.2 Support Triage Workflow

```mermaid
graph TD
    T[Ticket Created] --> A1[Classify via LLM]
    A1 --> C{Priority?}
    C -->|P1 Critical| ESC[Escalate to On-Call]
    C -->|P2 High| Q[Route to Senior Queue]
    C -->|P3 Medium| STD[Standard Queue]
    C -->|P4 Low| BOT[Auto-Response Bot]
    ESC --> N[Notify Management]
    Q --> N2[Assign Agent]
    STD --> N2
    BOT --> KB[Search Knowledge Base]
    KB --> REPLY[Auto-Reply with Article]
```

## 6. Workflow Monitoring

### 6.1 Prometheus Alert Rules

| Alert | Condition | Severity |
|-------|----------|----------|
| ActivepiecesWorkerSaturation | Busy ratio > 80% for 5m | Warning |
| TemporalTaskBacklog | Backlog > 1000 for 10m | Critical |
| KafkaLagHigh | Consumer lag > 5000 for 10m | Warning |
| ApiErrorRate | 5xx rate > 5% for 5m | Critical |
| TenantCostSpike | Cost increase > $100/hr for 15m | Info |

### 6.2 Grafana Dashboard Panels

The WaaS Overview dashboard (`config/grafana/dashboards/waas-overview.json`) includes:
- Workflow execution rate (per tenant, per type)
- Execution duration percentiles (p50, p95, p99)
- Error rate by workflow
- Active workflow count
- Temporal task queue depth
- Kafka consumer lag
- Connector latency heatmap
