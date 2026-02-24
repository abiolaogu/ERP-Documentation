# ERP-DBaaS High-Level Design

## Document Control

| Field             | Value                              |
|-------------------|------------------------------------|
| Document Title    | ERP-DBaaS High-Level Design        |
| Version           | 1.0.0                             |
| Date              | 2026-02-24                         |
| Classification    | Internal - Engineering             |
| Author            | Platform Engineering Team          |

---

## Table of Contents

1. [System Context Diagram](#1-system-context-diagram)
2. [Container Diagram](#2-container-diagram)
3. [Component Diagram per Engine Operator](#3-component-diagram-per-engine-operator)
4. [Data Flow Diagrams](#4-data-flow-diagrams)
5. [Integration Points with Other ERP Modules](#5-integration-points-with-other-erp-modules)

---

## 1. System Context Diagram

The system context diagram shows ERP-DBaaS as a black box and its interactions with external actors and systems.

```
                          ┌──────────────────┐
                          │   ERP Module      │
                          │   Developers      │
                          └────────┬─────────┘
                                   │ Provision / Manage DBs
                                   ▼
 ┌──────────────┐         ┌──────────────────┐         ┌──────────────┐
 │  Authentik   │────────>│                  │<────────│ Apache       │
 │  (Identity)  │  AuthN  │   ERP-DBaaS      │  Events │ Pulsar       │
 └──────────────┘  AuthZ  │   Platform       │         │ (Messaging)  │
                          │                  │         └──────────────┘
 ┌──────────────┐         │                  │         ┌──────────────┐
 │  Kubernetes  │<────────│                  │────────>│ RustFS       │
 │  Cluster(s)  │  CRDs   │                  │ Backups │ (Object      │
 └──────────────┘         └──────────────────┘         │  Storage)    │
                                   │                   └──────────────┘
                                   │ Metrics
                                   ▼
                          ┌──────────────────┐
                          │ VictoriaMetrics  │
                          │ (Monitoring)     │
                          └──────────────────┘
```

### External Actors and Systems

| Actor / System     | Interaction                                             |
|--------------------|---------------------------------------------------------|
| ERP Module Devs    | Provision, scale, backup, and manage database instances  |
| Authentik          | JWT-based authentication and RBAC authorization         |
| Kubernetes         | Runtime for database instances and operators             |
| Apache Pulsar      | Event bus for lifecycle notifications                    |
| RustFS             | Object storage for encrypted backups                     |
| VictoriaMetrics    | Metrics collection and alerting                          |

---

## 2. Container Diagram

The container diagram decomposes ERP-DBaaS into its major runtime containers.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           ERP-DBaaS Platform                            │
│                                                                         │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐     │
│  │  React Frontend  │    │  Go Gateway     │    │  Node.js API    │     │
│  │  (Refine.dev +   │───>│  (Port 8090)    │───>│  (dbaas-api)    │     │
│  │   Ant Design)    │    │                 │    │  (Port 3000)    │     │
│  │  Port 5173       │    │  - JWT Validate │    │                 │     │
│  └─────────────────┘    │  - Rate Limit   │    │  - REST API     │     │
│                          │  - Route        │    │  - Business     │     │
│                          └─────────────────┘    │    Logic        │     │
│                                                  │  - CRD Mgmt    │     │
│                                                  └────────┬────────┘     │
│                                                           │              │
│                          ┌────────────────────────────────┤              │
│                          │                                │              │
│                          ▼                                ▼              │
│  ┌─────────────────────────────┐    ┌──────────────────────────────┐    │
│  │  Hasura GraphQL Engine      │    │  Kubernetes Control Plane    │    │
│  │                             │    │                              │    │
│  │  - Auto-generated GraphQL   │    │  ┌────────────────────────┐  │    │
│  │  - Real-time subscriptions  │    │  │  Engine Operators (x8) │  │    │
│  │  - Permissions / Row-level  │    │  │                        │  │    │
│  │    security                 │    │  │  - YugabyteDB Operator │  │    │
│  └──────────┬──────────────────┘    │  │  - DragonflyDB Operator│  │    │
│             │                       │  │  - ClickHouse Operator │  │    │
│             ▼                       │  │  - Tembo Operator      │  │    │
│  ┌─────────────────────────────┐    │  │  - SurrealDB Operator  │  │    │
│  │  YugabyteDB (Registry)     │    │  │  - QuestDB Operator    │  │    │
│  │                             │    │  │  - Doris Operator      │  │    │
│  │  - Instance metadata        │    │  │  - InfluxDB Operator   │  │    │
│  │  - Backup catalog           │    │  └────────────────────────┘  │    │
│  │  - Quota ledger             │    │                              │    │
│  │  - Metering records         │    │  ┌────────────────────────┐  │    │
│  │  - Plugin registry          │    │  │  Backup Controller     │  │    │
│  └─────────────────────────────┘    │  │  Credential Manager    │  │    │
│                                      │  │  Metering Collector    │  │    │
│                                      │  └────────────────────────┘  │    │
│                                      └──────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### Container Responsibilities

| Container               | Technology       | Responsibility                                      |
|--------------------------|------------------|-----------------------------------------------------|
| React Frontend           | React + Refine   | User interface for instance management               |
| Go Gateway               | Go               | Authentication, rate limiting, request routing       |
| Node.js API (dbaas-api)  | Node.js          | Business logic, CRD lifecycle, quota enforcement     |
| Hasura GraphQL           | Hasura           | Auto-generated GraphQL API with real-time subs       |
| YugabyteDB (Registry)    | YugabyteDB       | Persistent storage for platform metadata             |
| Engine Operators (x8)    | Go               | Kubernetes operators for each database engine        |
| Backup Controller        | Go               | Scheduled and on-demand backup orchestration         |
| Credential Manager       | Go               | Credential generation, rotation, and revocation      |
| Metering Collector       | Go               | Resource usage collection for billing                |

---

## 3. Component Diagram per Engine Operator

Each engine operator follows a consistent internal architecture with engine-specific adapters.

```
┌─────────────────────────────────────────────────────┐
│                  Engine Operator (Generic)            │
│                                                       │
│  ┌──────────────┐    ┌──────────────┐                │
│  │  CRD Watcher  │───>│ Reconciler   │               │
│  │  (Informer)   │    │ Loop         │               │
│  └──────────────┘    └──────┬───────┘               │
│                              │                        │
│                   ┌──────────┼──────────┐            │
│                   ▼          ▼          ▼            │
│            ┌───────────┐ ┌────────┐ ┌──────────┐    │
│            │ Provision  │ │ Scale  │ │ Delete   │    │
│            │ Handler    │ │ Handler│ │ Handler  │    │
│            └─────┬─────┘ └───┬────┘ └────┬─────┘    │
│                  │           │           │           │
│                  ▼           ▼           ▼           │
│            ┌─────────────────────────────────────┐   │
│            │       Engine-Specific Adapter        │   │
│            │                                      │   │
│            │  - StatefulSet / Deployment spec     │   │
│            │  - Health check implementation       │   │
│            │  - Backup integration                │   │
│            │  - Connection string format          │   │
│            │  - Plugin support                    │   │
│            └─────────────────────────────────────┘   │
│                              │                        │
│                              ▼                        │
│            ┌─────────────────────────────────────┐   │
│            │         Status Reporter              │   │
│            │  - Update CRD status                 │   │
│            │  - Emit events to Pulsar             │   │
│            │  - Publish metrics                   │   │
│            └─────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Engine-Specific Adapter Details

| Engine      | Workload Type  | Storage      | HA Mode            | Plugin Support |
|-------------|----------------|--------------|---------------------|----------------|
| YugabyteDB  | StatefulSet    | PVC (SSD)    | Raft consensus      | PG extensions  |
| DragonflyDB | Deployment     | EmptyDir/PVC | Sentinel replication| None           |
| ClickHouse  | StatefulSet    | PVC (SSD)    | ReplicatedMergeTree | Dictionaries   |
| Tembo (PG)  | StatefulSet    | PVC (SSD)    | Streaming repl.     | PG extensions  |
| SurrealDB   | Deployment     | PVC          | TiKV cluster        | None           |
| QuestDB     | StatefulSet    | PVC (NVMe)   | Replication          | None           |
| Apache Doris| StatefulSet    | PVC (SSD)    | FE + BE groups      | UDF plugins    |
| InfluxDB    | StatefulSet    | PVC (SSD)    | Anti-entropy repl.  | Telegraf       |

---

## 4. Data Flow Diagrams

### 4.1 Instance Provisioning Data Flow

```
Developer ──[POST /instances]──> Go Gateway ──[JWT check]──> Node.js API
                                                                  │
                                        ┌─────────────────────────┤
                                        ▼                         ▼
                                 Quota Ledger              CRD Generator
                                 (YugabyteDB)                    │
                                        │                         ▼
                                        │                  K8s API Server
                                        │                         │
                                        │                         ▼
                                        │                 Engine Operator
                                        │                         │
                                        │                         ▼
                                        │              Database Instance +
                                        │              K8s Secret (creds)
                                        │                         │
                                        └──────── Metering ───────┘
```

### 4.2 Backup Data Flow

```
Backup Controller ──[snapshot]──> Database Instance
       │
       ▼
 Raw Dump Data ──[zstd compress]──> Compressed Data
       │
       ▼
 Compressed Data ──[AES-256-GCM encrypt]──> Encrypted Blob
       │
       ▼
 Encrypted Blob ──[PUT]──> RustFS (/backups/{tenant}/{instance}/{ts}.enc)
       │
       ▼
 Backup Metadata ──[INSERT]──> YugabyteDB (backup_catalog table)
```

### 4.3 Metrics Data Flow

```
 Database Instance ──[engine metrics]──> Metrics Exporter (sidecar)
       │
       ▼
 Prometheus Format ──[scrape]──> VictoriaMetrics
       │
       ▼
 Metering Collector ──[aggregate]──> Metering Records (YugabyteDB)
       │
       ▼
 Billing Pipeline ──[export]──> ERP-Billing Module
```

---

## 5. Integration Points with Other ERP Modules

### 5.1 Integration Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ ERP-Finance │     │ ERP-HRM     │     │ ERP-CRM     │
│             │     │             │     │             │
│ Needs:      │     │ Needs:      │     │ Needs:      │
│ YugabyteDB  │     │ YugabyteDB  │     │ SurrealDB   │
│ ClickHouse  │     │ DragonflyDB │     │ DragonflyDB │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       └───────────────────┴───────────────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │   ERP-DBaaS      │
                  │   GraphQL API    │
                  │   (Hasura)       │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │  Hasura          │
                  │  Federation      │
                  │  Gateway         │
                  └──────────────────┘
```

### 5.2 Integration Point Details

| ERP Module        | Integration Method        | Database Engines Used           | Data Exchange           |
|-------------------|---------------------------|---------------------------------|-------------------------|
| ERP-Finance       | GraphQL + REST API        | YugabyteDB, ClickHouse          | Provisioning, metering  |
| ERP-HRM           | GraphQL + REST API        | YugabyteDB, DragonflyDB         | Provisioning, creds     |
| ERP-CRM           | GraphQL + REST API        | SurrealDB, DragonflyDB          | Provisioning, creds     |
| ERP-Inventory     | GraphQL + REST API        | YugabyteDB, QuestDB             | Provisioning, backups   |
| ERP-Billing       | Event (Pulsar) + REST     | Metering data export            | Usage records           |
| ERP-IAM           | JWT / OIDC                | Authentik integration           | AuthN/AuthZ tokens      |
| ERP-Observability | Metrics (VictoriaMetrics) | All engines                     | Metrics, alerts         |
| Hasura Federation | Remote Schema             | GraphQL over dbaas-api          | Federated queries       |

### 5.3 Service Discovery

Database endpoints provisioned by ERP-DBaaS are discoverable by other ERP modules through:

1. **Kubernetes Secrets**: Connection strings injected as secrets in the consuming module namespace.
2. **Hasura Remote Schema**: GraphQL queries for instance metadata and connection info.
3. **Apache Pulsar Events**: `instance.provisioned` events carry connection details for automated consumption.
4. **DNS Service Records**: Kubernetes Services with predictable naming: `{instance-name}.{namespace}.svc.cluster.local`.

---

## Appendix: Technology Stack Summary

| Layer          | Technology                     | Purpose                         |
|----------------|--------------------------------|---------------------------------|
| Frontend       | React, Refine.dev, Ant Design  | User interface                  |
| Gateway        | Go                             | Auth, rate limiting, routing    |
| API            | Node.js                        | Business logic, REST endpoints  |
| GraphQL        | Hasura                         | Auto-generated GraphQL          |
| Operators      | Go (controller-runtime)        | K8s operator pattern            |
| Registry DB    | YugabyteDB                     | Platform metadata               |
| Object Storage | RustFS                         | Backup storage                  |
| Messaging      | Apache Pulsar                  | Event-driven communication      |
| Monitoring     | VictoriaMetrics                | Metrics and alerting            |
| Identity       | Authentik                      | Authentication and authorization|
