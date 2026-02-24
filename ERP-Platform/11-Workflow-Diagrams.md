# ERP-Platform Workflow Diagrams

> **Document ID:** ERP-PLAT-WF-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [10-Use-Cases.md](./10-Use-Cases.md), [04-Software-Architecture.md](./04-Software-Architecture.md)

---

## 1. Tenant Onboarding Workflow

```mermaid
flowchart TD
    A([Start: Sales Closes Deal]) --> B[Customer selects plan]
    B --> C{Plan Type?}
    C -->|Single| D[Select individual modules]
    C -->|Bundle| E[Choose curated bundle]
    C -->|Suite| F[Custom module combination]
    D --> G[Validate SKUs against catalog]
    E --> H[Resolve bundle to module SKUs]
    F --> G
    H --> I[Deduplicate resolved SKUs]
    I --> G
    G --> J{All SKUs valid?}
    J -->|No| K[Return error to admin]
    K --> B
    J -->|Yes| L[Create Subscription Record]
    L --> M[POST /v1/tenant-provisioner]
    M --> N[Create Tenant Record]
    N --> O[Seed Entitlements]
    O --> P[Configure Web Hosting]
    P --> Q[Register Modules in Registry]
    Q --> R[Initiate Health Checks]
    R --> S{All modules healthy?}
    S -->|No| T[Retry / Alert Admin]
    T --> R
    S -->|Yes| U[Send Welcome Notification]
    U --> V[Launch Activation Wizard]
    V --> W[Admin Completes Setup]
    W --> X([End: Tenant Active])

    L -.->|Event| AE1[erp.platform.subscription.created]
    N -.->|Event| AE2[erp.platform.tenant-provisioner.created]
    U -.->|Event| AE3[erp.platform.notification-hub.created]
    V -.->|Event| AE4[erp.platform.activation-wizard.created]
```

---

## 2. Subscription Lifecycle Workflow

```mermaid
stateDiagram-v2
    [*] --> PlanSelection: Customer initiates
    PlanSelection --> Validation: Submit subscription
    Validation --> Active: Validation passes
    Validation --> PlanSelection: Validation fails

    Active --> Upgrading: Request upgrade
    Upgrading --> Active: Upgrade complete

    Active --> Downgrading: Request downgrade
    Downgrading --> Active: Downgrade complete

    Active --> GracePeriod: Request cancellation
    GracePeriod --> Active: Reactivate (within 30 days)
    GracePeriod --> Cancelled: Grace period expires

    Active --> Suspended: Payment failure
    Suspended --> Active: Payment resolved
    Suspended --> Cancelled: Payment unresolved (60 days)

    Cancelled --> [*]

    note right of Active
        Entitlements active
        All subscribed modules accessible
        Audit logging enabled
    end note

    note right of GracePeriod
        30-day retention
        Read-only access
        Data export available
    end note
```

---

## 3. Module Activation Workflow

```mermaid
flowchart TD
    A([Start]) --> B[Admin requests module activation]
    B --> C[Check tenant subscription]
    C --> D{Module in entitlements?}
    D -->|No| E[Redirect to subscription upgrade]
    E --> F[Upgrade subscription]
    F --> D
    D -->|Yes| G[Query Module Registry]
    G --> H{Module registered?}
    H -->|No| I[Register module]
    I --> J[Configure module for tenant]
    H -->|Yes| J
    J --> K[Run health check]
    K --> L{Health check passed?}
    L -->|No| M[Wait 30 seconds]
    M --> N{Retry count < 5?}
    N -->|Yes| K
    N -->|No| O[Alert admin - activation failed]
    O --> P([End: Failed])
    L -->|Yes| Q[Activate module for tenant]
    Q --> R[Update entitlement-engine]
    R --> S[Send activation notification]
    S --> T[Log to audit service]
    T --> U([End: Module Active])
```

---

## 4. Marketplace Installation Workflow

```mermaid
flowchart TD
    A([Start]) --> B[Admin browses marketplace]
    B --> C[Select module listing]
    C --> D[Review module details]
    D --> E{Proceed with install?}
    E -->|No| B
    E -->|Yes| F[Check tenant entitlements]
    F --> G{Entitled?}
    G -->|No| H[Prompt subscription upgrade]
    H --> I{Upgrade accepted?}
    I -->|No| B
    I -->|Yes| J[Process upgrade]
    J --> G
    G -->|Yes| K[Download module package]
    K --> L[Verify package signature]
    L --> M{Signature valid?}
    M -->|No| N[Reject installation]
    N --> O[Log security event]
    O --> B
    M -->|Yes| P[Install module]
    P --> Q[Register in module-registry]
    Q --> R[Run health check]
    R --> S{Healthy?}
    S -->|No| T[Rollback installation]
    T --> U[Notify admin of failure]
    U --> B
    S -->|Yes| V[Activate module]
    V --> W[Update entitlements]
    W --> X[Send success notification]
    X --> Y[Log to audit service]
    Y --> Z([End: Module Installed])
```

---

## 5. Payment Processing Workflow

```mermaid
flowchart TD
    A([Billing Cycle Trigger]) --> B[Calculate invoice]
    B --> C[Determine subscribed modules and plan]
    C --> D[Apply pricing rules]
    D --> E[Generate invoice]
    E --> F[Send to payment gateway]
    F --> G{Payment successful?}
    G -->|Yes| H[Record payment]
    H --> I[Send receipt notification]
    I --> J[Update subscription status = Active]
    J --> K[Log to audit service]
    K --> L([End: Payment Complete])
    G -->|No| M[Retry payment]
    M --> N{Retry count < 3?}
    N -->|Yes| O[Wait 24 hours]
    O --> F
    N -->|No| P[Send payment failure notification]
    P --> Q[Suspend subscription]
    Q --> R{Payment resolved within 60 days?}
    R -->|Yes| S[Reactivate subscription]
    S --> J
    R -->|No| T[Cancel subscription]
    T --> U[Begin grace period]
    U --> K
```

---

## 6. Audit Trail Workflow

```mermaid
flowchart TD
    A([Platform Service performs CRUD]) --> B[Service generates CloudEvent]
    B --> C[Publish to NATS topic]
    C --> D{Topic pattern}
    D -->|erp.platform.*.created| E[Audit Service Consumer]
    D -->|erp.platform.*.updated| E
    D -->|erp.platform.*.deleted| E
    D -->|erp.platform.*.listed| E
    D -->|erp.platform.*.read| E
    E --> F[Validate event schema]
    F --> G{Schema valid?}
    G -->|No| H[Log schema violation]
    H --> I[Dead-letter queue]
    G -->|Yes| J[Enrich with metadata]
    J --> K[Persist to PostgreSQL]
    K --> L{Retention check}
    L -->|Within policy| M[Available for query]
    L -->|Expired| N[Archive to cold storage]
    M --> O[Queryable via GET /v1/audit]
    N --> P[Queryable via archive API]

    subgraph "Audit Event Schema"
        S1["event_id: UUID"]
        S2["event_topic: string"]
        S3["tenant_id: string"]
        S4["actor_id: string"]
        S5["payload: JSON"]
        S6["timestamp: ISO8601"]
        S7["correlation_id: string"]
    end
```

---

## 7. Incident Escalation Workflow

```mermaid
flowchart TD
    A([Incident Detected]) --> B{Detection Source}
    B -->|Health Check Failure| C[Module Registry Alert]
    B -->|Error Rate Spike| D[Prometheus Alert]
    B -->|Customer Report| E[Support Ticket]
    B -->|Audit Anomaly| F[Audit Service Alert]

    C & D & E & F --> G[Incident Created]
    G --> H{Severity Assessment}

    H -->|P4: Low| I[Assign to team queue]
    I --> J[Resolve within 5 business days]

    H -->|P3: Medium| K[Assign to on-call engineer]
    K --> L[Resolve within 24 hours]

    H -->|P2: High| M[Page on-call + tech lead]
    M --> N[Resolve within 4 hours]
    N --> O[Post-incident review]

    H -->|P1: Critical| P[Page on-call + tech lead + VP Eng]
    P --> Q[War room activated]
    Q --> R[Status page updated]
    R --> S[Resolve within 1 hour]
    S --> T[Post-mortem within 48 hours]

    J & L & O & T --> U[Update audit log]
    U --> V[Close incident]
    V --> W([End])
```

---

## 8. AIDD Guardrail Evaluation Workflow

```mermaid
flowchart TD
    A([AI Action Request]) --> B[Extract action metadata]
    B --> C[Calculate confidence score]
    C --> D{Confidence >= 0.70?}
    D -->|No| E[BLOCK action]
    E --> F[Log: blocked - low confidence]
    F --> G([End: Blocked])

    D -->|Yes| H{Confidence >= 0.82?}
    H -->|No| I[Queue for human review]
    I --> J{Human approves?}
    J -->|No| K[REJECT action]
    K --> L[Log: rejected by human]
    L --> G
    J -->|Yes| M[Continue evaluation]

    H -->|Yes| M
    M --> N{Blast radius <= 5000 records?}
    N -->|No| O[Queue for human review - blast radius]
    O --> J
    N -->|Yes| P{Financial value <= $100K?}
    P -->|No| Q[Queue for human review - high value]
    Q --> J
    P -->|Yes| R{High risk auto-execute enabled?}
    R -->|No - high risk detected| S[Queue for human review - risk]
    S --> J
    R -->|Yes / low risk| T[AUTO-APPROVE action]
    T --> U[Execute action]
    U --> V[Log: approved and executed]
    V --> W([End: Executed])
```

---

## 9. Catalog Update Workflow

```mermaid
flowchart TD
    A([Product team defines new module/bundle]) --> B[Update catalog/products.json]
    B --> C[Create pull request]
    C --> D[Automated validation]
    D --> E{JSON schema valid?}
    E -->|No| F[PR blocked - fix schema]
    F --> B
    E -->|Yes| G{SKU uniqueness check}
    G -->|Duplicate| H[PR blocked - fix SKU]
    H --> B
    G -->|Unique| I[Architecture review]
    I --> J{Approved?}
    J -->|No| K[Request changes]
    K --> B
    J -->|Yes| L[Merge to main]
    L --> M[CI/CD builds new images]
    M --> N[Deploy to staging]
    N --> O[Integration tests]
    O --> P{Tests pass?}
    P -->|No| Q[Rollback]
    P -->|Yes| R[Deploy to production]
    R --> S[Services reload catalog]
    S --> T[Verify catalog version]
    T --> U([End: Catalog Updated])
```

---

*For detailed use cases, see [10-Use-Cases.md](./10-Use-Cases.md). For architecture diagrams, see [04-Software-Architecture.md](./04-Software-Architecture.md).*
