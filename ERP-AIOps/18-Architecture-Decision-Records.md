# ERP-AIOps Architecture Decision Records

> **Document ID:** ERP-AIOPS-ADR-018
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [16-Changelog.md](./16-Changelog.md), [17-README.md](./17-README.md)

---

## Overview

This document captures the key Architecture Decision Records (ADRs) for the ERP-AIOps platform. Each ADR follows the format: Context, Decision, Status, Consequences. These decisions represent the foundational technical choices that shape the platform's architecture, performance characteristics, and operational model.

---

## Table of Contents

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| ADR-001 | Rust for Core Platform | Accepted | 2025-07-15 |
| ADR-002 | Python for AI Brain | Accepted | 2025-07-18 |
| ADR-003 | Axum over Actix-web | Accepted | 2025-07-20 |
| ADR-004 | Cargo Workspace with Modular Crates | Accepted | 2025-07-22 |
| ADR-005 | LLM-Powered Root Cause Analysis | Accepted | 2025-09-10 |
| ADR-006 | Adaptive Thresholds over Static Thresholds | Accepted | 2025-09-25 |
| ADR-007 | Topology-Aware Event Correlation | Accepted | 2025-10-15 |
| ADR-008 | AIDD Stack Compliance | Accepted | 2025-07-12 |

---

## ADR-001: Rust for Core Platform

**Date:** 2025-07-15
**Status:** Accepted
**Deciders:** AIOps Architecture Board, CTO

### Context

ERP-AIOps is an operations platform that processes high volumes of telemetry data (metrics, logs, traces, events) in real time and must deliver low-latency responses for incident detection, anomaly analysis, and remediation orchestration. The core platform handles:

- **High-throughput event ingestion**: Up to 100,000 events/second per tenant at peak.
- **Real-time anomaly detection**: Statistical computations on streaming time-series data with sub-100ms latency requirements.
- **Event correlation**: Complex event processing across multiple event streams with temporal and topological constraints.
- **Remediation orchestration**: Safety-critical execution of remediation actions where bugs can cause production outages.
- **Multi-tenant isolation**: Strict data isolation across tenants sharing the same infrastructure.

The language choice for the core platform must optimize for: performance (throughput and latency), reliability (memory safety, no undefined behavior), concurrency (async I/O for network-heavy workloads), and operational efficiency (low resource consumption per tenant).

**Languages Considered:**

| Language | Strengths | Weaknesses (for this use case) |
|----------|-----------|-------------------------------|
| **Rust** | Memory safety without GC, zero-cost abstractions, excellent async runtime (Tokio), predictable latency, low memory footprint | Steeper learning curve, longer compilation times |
| **Go** | Fast compilation, goroutines, large ecosystem, team familiarity | GC pauses (problematic for P99 latency), higher memory usage per connection, less expressive type system |
| **Java/Kotlin** | Mature ecosystem, JVM optimization, excellent tooling | JVM startup time, GC pauses, high memory overhead |
| **C++** | Maximum performance, zero overhead | Memory safety issues, undefined behavior risks, slower development velocity |

### Decision

**Use Rust as the primary language for the ERP-AIOps core platform.**

Specific justifications:

1. **No Garbage Collection Pauses**: AIOps real-time event processing requires predictable P99 latencies. Rust's ownership model eliminates GC pauses entirely, providing consistent sub-millisecond response times for the event correlation and anomaly detection hot paths. In benchmarks, Rust event processing showed P99 latency of 0.8ms vs. Go's 12ms (due to GC pauses) for identical workloads.

2. **Memory Safety Without Runtime Cost**: AIOps remediation actions can restart services, scale infrastructure, and modify DNS records. Memory bugs (use-after-free, buffer overflows, data races) in this context could cause catastrophic production failures. Rust's borrow checker eliminates entire classes of memory bugs at compile time with zero runtime overhead.

3. **Efficient Multi-Tenant Resource Usage**: Each tenant's event processing pipeline runs concurrently. Rust's zero-cost abstractions and minimal memory footprint (approximately 2MB per tenant pipeline vs. 15MB in Go) enable supporting 10x more tenants per node, directly reducing infrastructure costs.

4. **Fearless Concurrency**: Rust's type system prevents data races at compile time. The event correlation engine processes events from multiple streams concurrently, and Rust ensures that shared state is accessed safely without requiring runtime locks in most cases.

5. **Ecosystem Maturity**: The Rust async ecosystem (Tokio, Tower, Axum, sqlx, tonic) has reached production maturity. Crates for time-series processing, statistical computation, and graph algorithms are available and well-maintained.

6. **SIMD and Low-Level Optimization**: Anomaly detection algorithms benefit from SIMD (Single Instruction, Multiple Data) optimizations for batch statistical computations. Rust provides safe SIMD abstractions via `std::simd` and `packed_simd` crates.

### Consequences

**Positive:**
- Predictable P99 latencies under 1ms for event processing hot paths.
- Memory safety guarantees eliminate entire classes of production incidents.
- 10x improvement in tenant density per node compared to GC-based languages.
- Compile-time concurrency safety reduces production data race bugs to zero.
- Smaller container images (approximately 30MB static binary vs. 200MB+ for JVM).

**Negative:**
- Steeper learning curve for developers without Rust experience. Mitigation: dedicated Rust onboarding program, pair programming, and comprehensive code review.
- Longer compilation times (full workspace: approximately 8 minutes, incremental: approximately 30 seconds). Mitigation: aggressive use of `cargo check`, `sccache`, and incremental compilation.
- Smaller talent pool compared to Go or Java. Mitigation: hire for strong systems programming fundamentals and train on Rust-specific idioms.
- Some third-party library gaps compared to Go/Java ecosystems. Mitigation: contribute to and maintain internal crates where needed.

### Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Team inability to learn Rust | Low | High | 4-week Rust bootcamp, pair programming, mentorship program |
| Compilation time slows development | Medium | Medium | `sccache`, workspace partitioning, CI caching, `cargo check` workflow |
| Library ecosystem gaps | Low | Medium | Maintain internal crate registry, contribute upstream |
| Debugging complexity | Medium | Low | `tracing` crate for structured logging, Tokio console for async debugging |

---

## ADR-002: Python for AI Brain

**Date:** 2025-07-18
**Status:** Accepted
**Deciders:** AIOps Architecture Board, AI/ML Lead

### Context

ERP-AIOps requires an ML inference service (the "AI Brain") that provides:

- Anomaly detection using statistical and ML models (Isolation Forest, Autoencoders, LSTM).
- Root cause analysis using Large Language Models (GPT-4, Claude, LLaMA).
- Time-series forecasting using Prophet, ARIMA, and neural forecasting models.
- Event correlation using graph neural networks.
- Cost anomaly detection using ensemble methods.

The AI Brain must support rapid model iteration, integrate with the ML research ecosystem, and serve inference requests with acceptable latency (P99 under 500ms for most operations).

**Languages Considered:**

| Language | Strengths | Weaknesses (for this use case) |
|----------|-----------|-------------------------------|
| **Python** | Dominant ML ecosystem (PyTorch, scikit-learn, transformers, prophet), rapid prototyping, LLM integration libraries | Slower execution, GIL for CPU-bound tasks |
| **Rust** | Performance, already used for core | Immature ML ecosystem, slow model iteration cycle |
| **Julia** | Fast numerical computing, ML capabilities | Small ecosystem, deployment complexity, small talent pool |

### Decision

**Use Python (FastAPI) as the language and framework for the AI Brain service.**

Specific justifications:

1. **ML Ecosystem Dominance**: Python is the uncontested leader in ML/AI tooling. All major ML frameworks (PyTorch, TensorFlow, scikit-learn), LLM libraries (LangChain, OpenAI SDK, Anthropic SDK, Hugging Face Transformers), forecasting tools (Prophet, statsmodels, NeuralProphet), and inference runtimes (ONNX Runtime, TensorRT) are Python-first.

2. **Rapid Model Iteration**: ML model development is inherently experimental. Python's dynamic typing, Jupyter notebook integration, and REPL-driven development enable data scientists to iterate on models 5-10x faster than in Rust.

3. **FastAPI for Production Serving**: FastAPI provides automatic OpenAPI documentation, Pydantic-based request validation, async request handling, and excellent performance for a Python framework (comparable to Node.js via Starlette/uvicorn).

4. **LLM Integration**: Root cause analysis requires integration with multiple LLM providers. Python's LangChain, OpenAI, and Anthropic libraries provide production-ready abstractions for prompt management, chain-of-thought reasoning, and output parsing.

5. **GPU Acceleration**: Python's ML frameworks provide seamless GPU acceleration via CUDA, which is critical for neural network inference (autoencoders, LSTM, GNN) and LLM operations.

### Consequences

**Positive:**
- Access to the entire ML/AI ecosystem without wrapper libraries.
- Data scientists and ML engineers can contribute models directly.
- Rapid model iteration and experimentation cycles.
- Pre-built LLM integration libraries reduce development effort.

**Negative:**
- Python's performance is lower than Rust for CPU-bound tasks. Mitigation: compute-heavy operations use NumPy/ONNX Runtime (C/C++ backends), not pure Python.
- GIL limits true parallelism for CPU-bound tasks. Mitigation: use multiprocessing for CPU-bound tasks, async for I/O-bound tasks, and offload heavy computation to ONNX Runtime.
- Additional service to deploy and monitor. Mitigation: comprehensive health checks, auto-scaling, and circuit breaker in Go Gateway.
- Language boundary between Rust and Python. Mitigation: gRPC with Protocol Buffers for typed, high-performance inter-service communication.

### Decision Boundary

The Rust core handles all data ingestion, storage, event processing, API serving, and orchestration. The Python AI Brain is called only for ML inference tasks that require the Python ML ecosystem. The Rust core is the system of record; the AI Brain is a stateless inference service.

---

## ADR-003: Axum over Actix-web

**Date:** 2025-07-20
**Status:** Accepted
**Deciders:** AIOps Architecture Board, Rust Tech Lead

### Context

The Rust API server requires an HTTP framework that supports:

- Async request handling via Tokio.
- Middleware composition (authentication, logging, tracing, rate limiting, CORS).
- Type-safe request extraction (path parameters, query strings, JSON bodies, headers).
- WebSocket support for real-time event streaming.
- HTTP/2 and gRPC support for AI Brain communication.
- Integration with the Tower middleware ecosystem.

**Frameworks Considered:**

| Framework | Strengths | Weaknesses |
|-----------|-----------|------------|
| **Axum** | Tower ecosystem integration, type-safe extractors, maintained by Tokio team, composable routing | Younger ecosystem, fewer tutorials |
| **Actix-web** | Mature, battle-tested, high benchmark performance, large community | Actor model complexity, custom middleware system (not Tower), historical maintainer controversy |
| **Warp** | Composable filter system, type safety | Complex type errors, less intuitive API for newcomers |
| **Rocket** | Ergonomic API, strong typing | Slower async adoption, smaller async ecosystem integration |

### Decision

**Use Axum as the HTTP framework for the Rust API server.**

Specific justifications:

1. **Tower Ecosystem Integration**: Axum is built on Tower, the standard middleware framework in the Rust ecosystem. This provides access to a rich library of production-ready middleware: `tower-http` (CORS, compression, tracing, request ID, timeout), `tower-limit` (rate limiting), `tower-retry` (retry policies). This composability means we do not need to implement common middleware from scratch.

2. **Maintained by the Tokio Team**: Axum is maintained by the same team that maintains Tokio, ensuring deep integration with the async runtime. API design decisions in Axum are coordinated with Tokio's roadmap, reducing the risk of compatibility issues.

3. **Type-Safe Extractors**: Axum's extractor system uses Rust's type system to parse and validate request data at compile time. A handler function's signature directly declares its dependencies:
   ```rust
   async fn create_incident(
       State(db): State<DbPool>,
       TenantId(tenant): TenantId,
       Json(body): Json<CreateIncidentRequest>,
   ) -> Result<Json<Incident>, ApiError> { ... }
   ```
   Invalid extractor combinations are caught at compile time, not at runtime.

4. **Composable Routing**: Axum's `Router` supports nested routing, route-specific middleware, and fallback handlers. This maps well to the AIOps API structure:
   ```rust
   Router::new()
       .nest("/api/v1/incidents", incident_routes())
       .nest("/api/v1/anomalies", anomaly_routes())
       .nest("/api/v1/topology", topology_routes())
       .layer(auth_middleware)
       .layer(tracing_middleware)
   ```

5. **WebSocket and SSE Support**: Built-in support for WebSocket upgrades and Server-Sent Events, which are used for real-time incident status streaming and anomaly notification.

6. **gRPC Compatibility**: Axum and Tonic (Rust gRPC) both use Tower, enabling them to share middleware and run on the same server (multiplexing HTTP and gRPC on a single port).

### Consequences

**Positive:**
- Access to the Tower middleware ecosystem reduces custom middleware development.
- Compile-time validation of request extraction eliminates runtime type errors.
- Unified middleware stack across HTTP and gRPC services.
- Strong alignment with the broader Tokio/Tower ecosystem direction.

**Negative:**
- Axum's compile errors for invalid extractors can be verbose. Mitigation: use `axum-macros` for `#[debug_handler]` attribute that provides clearer error messages.
- Less community content (tutorials, blog posts) compared to Actix-web. Mitigation: maintain internal Axum cookbook and onboarding guide.

---

## ADR-004: Cargo Workspace with Modular Crates

**Date:** 2025-07-22
**Status:** Accepted
**Deciders:** AIOps Architecture Board, Rust Tech Lead

### Context

ERP-AIOps encompasses multiple subsystems: incident management, anomaly detection, root cause analysis, event correlation, topology mapping, auto-remediation, cost optimization, security scanning, forecasting, and supporting infrastructure (caching, metrics, tracing, configuration, authentication). The codebase is expected to grow to 40+ modules and 100,000+ lines of Rust code.

The code organization strategy must support:

- **Independent development**: Teams can work on separate subsystems without merge conflicts.
- **Selective compilation**: Developers can build and test only the crates they are working on.
- **Clear dependency boundaries**: Each subsystem's dependencies are explicit and minimal.
- **Reusable components**: Common functionality (database, caching, auth, models) is shared without duplication.
- **Compile-time optimization**: Parallel compilation of independent crates.

**Approaches Considered:**

| Approach | Strengths | Weaknesses |
|----------|-----------|------------|
| **Cargo Workspace (many crates)** | Clear boundaries, parallel compilation, selective testing, explicit dependencies | More Cargo.toml files to maintain, cross-crate refactoring is harder |
| **Single crate with modules** | Simple setup, easy refactoring | Long compilation times, no selective compilation, unclear boundaries, all code in one compilation unit |
| **Separate repositories** | Strong isolation, independent release cycles | Complex dependency management, cross-repo changes are painful, version coordination overhead |

### Decision

**Organize the Rust codebase as a Cargo workspace with 40+ modular crates, each representing a bounded context or infrastructure concern.**

**Workspace Structure:**

```
crates/
├── aiops-core/          # Core types, traits, error types, shared utilities
├── aiops-models/        # Data models (Incident, Anomaly, Rule, etc.)
├── aiops-config/        # Configuration loading and validation
├── aiops-db/            # Database connection pool, query helpers, migrations
├── aiops-cache/         # DragonflyDB caching abstractions
├── aiops-auth/          # JWT validation, RBAC, tenant extraction
├── aiops-api/           # Axum HTTP server, route composition, middleware
├── aiops-incidents/     # Incident management business logic
├── aiops-anomaly/       # Anomaly detection algorithms and engine
├── aiops-rca/           # Root cause analysis orchestration
├── aiops-correlation/   # Event correlation engine
├── aiops-topology/      # Topology graph management
├── aiops-remediation/   # Auto-remediation framework
├── aiops-cost/          # Cost optimization engine
├── aiops-security/      # Security scanning engine
├── aiops-forecasting/   # Forecasting and capacity planning
├── aiops-rules/         # Rules engine and policy evaluation
├── aiops-webhooks/      # Webhook delivery and retry management
├── aiops-ai-client/     # AI Brain gRPC/REST client
├── aiops-storage/       # RustFS/S3 object storage client
├── aiops-metrics/       # Prometheus metrics exposition
├── aiops-tracing/       # OpenTelemetry tracing setup
├── aiops-migrate/       # Database migration runner
└── ...                  # Additional infrastructure and domain crates
```

**Dependency Rules:**

1. Domain crates (incidents, anomaly, etc.) depend on `aiops-core`, `aiops-models`, and `aiops-db`.
2. Domain crates do NOT depend on each other directly. Cross-domain communication uses the event bus or API calls.
3. `aiops-api` depends on all domain crates and composes them into the HTTP API.
4. Infrastructure crates (cache, auth, metrics, tracing) are shared utilities with minimal dependencies.

### Consequences

**Positive:**
- Incremental compilation: changing `aiops-anomaly` only recompiles `aiops-anomaly` and `aiops-api`, not `aiops-incidents` or `aiops-correlation`.
- Selective testing: `cargo test -p aiops-anomaly` runs only anomaly detection tests in approximately 5 seconds instead of the full test suite in approximately 3 minutes.
- Clear API boundaries between subsystems, enforced by Rust's module visibility rules.
- Parallel compilation of independent crates utilizes all CPU cores.
- New subsystems are added by creating a new crate without modifying existing crates.

**Negative:**
- More Cargo.toml files to maintain (40+). Mitigation: workspace-level dependency declarations using `[workspace.dependencies]`.
- Cross-crate type sharing requires careful API design. Mitigation: `aiops-models` crate serves as the shared type registry.
- Initial setup is more complex. Mitigation: documented crate creation checklist in [19-CONTRIBUTING.md](./19-CONTRIBUTING.md).

---

## ADR-005: LLM-Powered Root Cause Analysis

**Date:** 2025-09-10
**Status:** Accepted
**Deciders:** AIOps Architecture Board, AI/ML Lead, CTO

### Context

Root Cause Analysis (RCA) is one of the most valuable but also most challenging capabilities in AIOps. Traditional RCA approaches include:

1. **Rule-Based RCA**: Predefined decision trees mapping symptoms to causes. Limited to known failure modes, brittle in the face of novel incidents.
2. **Statistical RCA**: Correlation analysis and causal inference on metric data. Effective for metric-level analysis but cannot reason about logs, configurations, or architectural context.
3. **Graph-Based RCA**: Dependency graph traversal to identify failing upstream services. Useful for identifying propagation paths but cannot determine the root cause within a failing service.

None of these approaches can synthesize heterogeneous evidence (metrics, logs, traces, configuration changes, deployment events) into a coherent narrative explaining why a failure occurred and what should be done about it.

### Decision

**Implement LLM-powered Root Cause Analysis that uses Large Language Models to synthesize heterogeneous evidence into natural-language root cause hypotheses, augmented by traditional statistical and graph-based methods.**

**Architecture:**

```
┌─────────────────────────────────────────────────┐
│              RCA Orchestrator (Rust)             │
│                                                  │
│  1. Collect Evidence                             │
│     ├─ Incident timeline                         │
│     ├─ Related anomalies (from anomaly engine)   │
│     ├─ Topology context (from topology engine)   │
│     ├─ Recent deployments/changes                │
│     ├─ Relevant log snippets                     │
│     └─ Similar historical incidents              │
│                                                  │
│  2. Pre-Analysis (Statistical)                   │
│     ├─ Granger causality on correlated metrics   │
│     ├─ Change-point detection on timelines       │
│     └─ Graph traversal for impact path           │
│                                                  │
│  3. LLM Analysis (via AI Brain)                  │
│     ├─ Structured prompt with evidence context   │
│     ├─ Chain-of-thought reasoning                │
│     └─ Hypothesis generation with confidence     │
│                                                  │
│  4. Post-Processing                              │
│     ├─ Hypothesis ranking                        │
│     ├─ Evidence linking                          │
│     └─ RCA report generation                     │
└─────────────────────────────────────────────────┘
```

**LLM Provider Strategy:**

| Provider | Use Case | Fallback |
|----------|----------|----------|
| Self-hosted LLaMA 3 (70B) | Default for all tenants, no data leaves infrastructure | N/A |
| OpenAI GPT-4 | Enhanced analysis when tenant opts in | Claude 3.5 |
| Anthropic Claude 3.5 | Alternative provider, used for consensus | GPT-4 |

**Prompt Engineering Principles:**

- Structured prompts with clear sections: INCIDENT_CONTEXT, ANOMALY_DATA, TOPOLOGY, RECENT_CHANGES, HISTORICAL_SIMILAR.
- Chain-of-thought reasoning: instruct the model to reason step-by-step through evidence.
- Confidence calibration: model rates its own confidence (0.0-1.0) for each hypothesis.
- Output schema enforcement: LLM output is constrained to a JSON schema for reliable parsing.

### Consequences

**Positive:**
- RCA quality dramatically higher than rule-based or statistical methods alone.
- Natural-language explanations are immediately actionable by operators without deep domain expertise.
- Synthesizes heterogeneous evidence types (metrics + logs + config + topology) that no single traditional method can handle.
- Continuously improves via operator feedback loop.

**Negative:**
- LLM inference cost per RCA analysis (approximately $0.05-0.50 depending on provider and context size). Mitigation: use self-hosted LLaMA for routine analysis, reserve commercial LLMs for high-severity incidents.
- Potential for LLM hallucination (plausible but incorrect hypotheses). Mitigation: always present multiple hypotheses with confidence scores, require operator confirmation before acting on RCA output.
- Latency of LLM inference (2-10 seconds). Mitigation: RCA is async; results are streamed to the operator as they are generated.
- Data privacy concerns with external LLM providers. Mitigation: default to self-hosted LLM; PII redaction before external API calls.

---

## ADR-006: Adaptive Thresholds over Static Thresholds

**Date:** 2025-09-25
**Status:** Accepted
**Deciders:** AIOps Architecture Board, SRE Lead

### Context

Traditional monitoring systems use static thresholds for anomaly detection (e.g., "alert if CPU > 80%"). This approach has well-documented problems:

1. **False Positives**: Static thresholds do not account for normal variation. A service with CPU usage that normally fluctuates between 60-85% will constantly trigger a static 80% threshold.
2. **False Negatives**: A service that normally runs at 20% CPU experiencing a gradual increase to 60% (3x normal) will not trigger a static 80% threshold, despite indicating a serious issue.
3. **Seasonal Blindness**: Static thresholds cannot distinguish between a legitimate weekday peak and an anomalous weekday peak. Monday morning traffic spikes are normal; Monday 3 AM spikes are not.
4. **Manual Maintenance**: Static thresholds require manual tuning per metric, per service, per environment. For an ERP platform with 20+ modules and thousands of metrics, this is operationally infeasible.

### Decision

**Implement adaptive thresholds as the default anomaly detection strategy, using dynamic baselines computed from historical data with seasonal decomposition.**

**Adaptive Threshold Algorithm:**

```
For each metric M at time T:

1. Compute baseline:
   - Exponential Moving Average (EMA) over the past 7 days, weighted by recency
   - Seasonal decomposition: extract daily, weekly patterns via STL decomposition
   - Baseline(T) = EMA(T) + Seasonal_Component(T)

2. Compute dynamic bounds:
   - Standard deviation σ computed over the same historical window
   - Upper bound = Baseline(T) + k * σ  (k = 3 by default, configurable)
   - Lower bound = Baseline(T) - k * σ

3. Detect anomaly:
   - If M(T) > Upper bound or M(T) < Lower bound:
     - Anomaly score = |M(T) - Baseline(T)| / σ  (normalized deviation)
     - Persist anomaly if score > threshold and duration > minimum_duration

4. Continuously update:
   - Update EMA and σ with each new data point
   - Exclude confirmed anomalies from baseline updates (anomaly-aware smoothing)
```

**Multi-Algorithm Ensemble:**

| Algorithm | Purpose | Implementation |
|-----------|---------|----------------|
| Adaptive EMA + STL | Primary detection for gradual and seasonal anomalies | Rust (`aiops-anomaly`) |
| Modified Z-Score (MAD) | Robust detection resistant to outlier contamination | Rust (`aiops-anomaly`) |
| Isolation Forest | Multivariate anomaly detection | Python AI Brain |
| LSTM Autoencoder | Sequence-based anomaly detection for complex patterns | Python AI Brain |
| DBSCAN | Density-based clustering for identifying anomalous clusters | Rust (`aiops-anomaly`) |

### Consequences

**Positive:**
- False positive rate reduced by 70-85% compared to static thresholds (measured in internal testing).
- False negative rate reduced by 60% -- gradual degradation and subtle anomalies are now detected.
- Zero manual threshold configuration required -- baselines are learned automatically.
- Seasonal awareness eliminates alerts during known traffic patterns (weekday peaks, monthly batch jobs, etc.).
- Operators focus on genuine anomalies instead of threshold noise.

**Negative:**
- Initial learning period of 7-14 days before adaptive thresholds are calibrated. Mitigation: use static thresholds as fallback during the learning period.
- Higher computational cost than static threshold comparison. Mitigation: SIMD-accelerated statistical computations in Rust provide sub-millisecond per-metric evaluation.
- Model drift if workload patterns change fundamentally. Mitigation: drift detection monitors baseline stability and triggers re-learning when patterns shift.
- Complexity in explaining anomaly scores to operators. Mitigation: anomaly detail view shows historical baseline, bounds, and the specific deviation.

---

## ADR-007: Topology-Aware Event Correlation

**Date:** 2025-10-15
**Status:** Accepted
**Deciders:** AIOps Architecture Board, SRE Lead, AI/ML Lead

### Context

A single infrastructure incident (e.g., database failover) can generate hundreds of alerts across dependent services: API errors, timeout alerts, queue backlog alerts, health check failures. Without correlation, operators are flooded with noise and cannot identify the actual root cause.

**Correlation strategies:**

| Strategy | Effectiveness | Limitations |
|----------|--------------|-------------|
| **Time-based** | Groups events within a time window | Cannot distinguish causally related events from coincidental ones |
| **Rule-based** | High precision for known patterns | Cannot handle novel failure modes |
| **Topology-aware** | Groups events based on service dependencies | Requires accurate topology data |
| **ML-based** | Can learn complex correlation patterns | Requires labeled training data |

### Decision

**Implement topology-aware event correlation as the primary correlation strategy, augmented by temporal, rule-based, and ML-based correlation.**

**Correlation Pipeline:**

```
Events In ──▶ Deduplication ──▶ Enrichment ──▶ Correlation ──▶ Groups Out
                                     │              │
                                     │              ├── Temporal (time window)
                                     │              ├── Topological (dependency graph)
                                     │              ├── Rule-based (CEP patterns)
                                     │              └── ML-based (GNN correlation)
                                     │
                                     └── Topology context
                                         Service dependencies
                                         Health propagation
```

**Topology-Aware Correlation Algorithm:**

```
Given an event E from service S:

1. Retrieve the dependency subgraph for S from the topology engine:
   - Upstream dependencies (services S depends on)
   - Downstream dependents (services that depend on S)

2. Query recent events from all services in the subgraph:
   - Time window: [E.timestamp - 10 minutes, E.timestamp + 2 minutes]

3. Score correlation candidates:
   - Topological distance: events from services closer in the dependency graph score higher
   - Temporal proximity: events closer in time score higher
   - Semantic similarity: events with similar descriptions/types score higher (via embeddings)
   - Causal direction: events from upstream services score higher as potential root causes

4. Group correlated events:
   - Primary event: the earliest event from the most upstream service in the correlated set
   - Contributing events: all other correlated events, ordered by topological distance
   - Root cause hypothesis: "Failure in {primary_service} propagated to {N} downstream services"
```

### Consequences

**Positive:**
- Alert volume reduced by 60-80% through intelligent grouping of related alerts.
- Root cause identification is accelerated because correlated groups already point to the most upstream failing service.
- Operators see a single correlated incident instead of dozens of independent alerts.
- Topology-aware correlation handles cascading failures that time-only correlation would miss (e.g., a slow database causing timeout errors in multiple services at different times).

**Negative:**
- Requires accurate and up-to-date topology data. Mitigation: auto-discovery from OpenTelemetry traces with manual override capability.
- Graph traversal has computational cost proportional to topology size. Mitigation: pre-computed adjacency lists and bounded traversal depth (max 5 hops).
- Novel services not yet in the topology graph cannot be correlated topologically. Mitigation: fall back to temporal and ML-based correlation for unknown services.

---

## ADR-008: AIDD Stack Compliance

**Date:** 2025-07-12
**Status:** Accepted
**Deciders:** CTO, VP Engineering, AIOps Architecture Board

### Context

The organization has adopted the AIDD (AI-Driven Development) stack as the standard technology platform for all new ERP modules. The AIDD stack mandates specific technology choices to ensure consistency, interoperability, and operational efficiency across the ERP ecosystem.

**AIDD Stack Requirements:**

| Layer | AIDD Standard | Purpose |
|-------|--------------|---------|
| Primary Language | Go or Rust | Performance, type safety |
| AI/ML Layer | Python (FastAPI) | ML ecosystem access |
| API Gateway | Go | Unified routing and protocol translation |
| Primary Database | YugabyteDB | Distributed SQL, horizontal scaling |
| Cache | DragonflyDB | Redis-compatible, high-performance caching |
| Object Storage | RustFS | S3-compatible, self-hosted object storage |
| Frontend | Refine.dev + Ant Design + Vite | Consistent admin UX across modules |
| Event Streaming | NATS / Redpanda | Async event-driven communication |
| Authentication | Authentik | Centralized SSO and RBAC |
| Observability | OpenTelemetry + Prometheus + Grafana | Unified telemetry |
| Container Runtime | Docker + Kubernetes | Deployment and orchestration |
| Multi-Tenancy | `tenant_id TEXT` column in all tables | Data isolation |

### Decision

**ERP-AIOps fully complies with the AIDD stack specification, with Rust as the primary language (permitted as an AIDD stack option alongside Go).**

**AIDD Compliance Matrix:**

| AIDD Requirement | ERP-AIOps Implementation | Compliant |
|-----------------|--------------------------|-----------|
| Primary Language | Rust 1.76+ (Axum) | Yes |
| AI/ML Layer | Python 3.12 (FastAPI) | Yes |
| API Gateway | Go 1.22 (custom gateway) | Yes |
| Primary Database | YugabyteDB | Yes |
| Cache | DragonflyDB | Yes |
| Object Storage | RustFS | Yes |
| Frontend | Refine.dev + Ant Design + Vite | Yes |
| Authentication | Authentik (JWT + RBAC) | Yes |
| Observability | OpenTelemetry + Prometheus + Grafana | Yes |
| Multi-Tenancy | `tenant_id TEXT` in all tables | Yes |
| Container Runtime | Docker + Kubernetes (Helm) | Yes |

**Deviation:** ERP-AIOps uses Rust instead of Go for the core platform. This is an approved deviation per ADR-001, as the AIDD stack permits both Go and Rust for primary services. The Go gateway satisfies the AIDD requirement for Go in the API gateway layer.

### Consequences

**Positive:**
- Full interoperability with other ERP modules via shared database patterns (`tenant_id`), authentication (Authentik JWT), and event streaming (NATS/Redpanda).
- Consistent operational practices: same monitoring (Prometheus/Grafana), same deployment model (Helm/Kubernetes), same database operations (YugabyteDB tooling).
- Frontend consistency: Refine.dev with purple theme (#7c3aed) provides familiar UX for operators already using other ERP modules.
- Shared infrastructure: DragonflyDB, RustFS, and Authentik instances are shared across ERP modules, reducing operational overhead.

**Negative:**
- Some AIDD stack choices may not be optimal for AIOps-specific workloads (e.g., a specialized time-series database like TimescaleDB or InfluxDB might outperform YugabyteDB for time-series queries). Mitigation: use YugabyteDB with time-series optimized schemas (BRIN indexes, time-range partitioning) and DragonflyDB for hot time-series caching.
- Standardization limits innovation in infrastructure choices. Mitigation: propose stack additions through the AIDD Architecture Review Board if a compelling case exists.

---

## ADR Lifecycle

### Statuses

| Status | Meaning |
|--------|---------|
| **Proposed** | Under discussion, not yet decided |
| **Accepted** | Decision made and being implemented |
| **Deprecated** | Decision superseded by a newer ADR |
| **Superseded** | Replaced by another ADR (linked) |

### Proposing a New ADR

1. Create a new section in this document following the template: Context, Decision, Consequences.
2. Assign the next sequential ADR number.
3. Set status to **Proposed**.
4. Present at the AIOps Architecture Board meeting (biweekly).
5. After approval, update status to **Accepted**.

---

*For questions about architectural decisions, contact the AIOps Architecture Board or raise a discussion in `#erp-aiops-architecture`.*
