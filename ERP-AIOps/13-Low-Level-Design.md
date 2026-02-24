# ERP-AIOps Low-Level Design

> **Document ID:** ERP-AIOPS-LLD-013
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [12-High-Level-Design.md](./12-High-Level-Design.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## 1. Rust Crate Architecture

The Rust API core is organized into 15 crates following a layered architecture. Each crate has well-defined dependencies enforced by the Cargo workspace.

### Crate Dependency Graph

```
┌─────────────────────────────────────────────────────────────────────┐
│                        APPLICATION LAYER                           │
│                                                                     │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐      │
│  │ aiops-api │  │ aiops-ws  │  │ aiops-cli │  │ aiops-    │      │
│  │ (Axum     │  │ (WebSocket│  │ (Admin    │  │ worker    │      │
│  │  routes)  │  │  server)  │  │  tooling) │  │ (bg jobs) │      │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘      │
│        │              │              │              │              │
├────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│        │         SERVICE LAYER       │              │              │
│        v              v              v              v              │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │ │
│  │  │ aiops-      │ │ aiops-      │ │ aiops-      │           │ │
│  │  │ incident    │ │ correlation │ │ rules       │           │ │
│  │  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘           │ │
│  │         │               │               │                   │ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │ │
│  │  │ aiops-      │ │ aiops-      │ │ aiops-      │           │ │
│  │  │ topology    │ │ remediation │ │ cost        │           │ │
│  │  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘           │ │
│  │         │               │               │                   │ │
│  │  ┌─────────────┐ ┌─────────────┐                           │ │
│  │  │ aiops-      │ │ aiops-      │                           │ │
│  │  │ security    │ │ forecast    │                           │ │
│  │  └──────┬──────┘ └──────┬──────┘                           │ │
│  └─────────┼───────────────┼───────────────────────────────────┘ │
│            │               │                                     │
├────────────┼───────────────┼─────────────────────────────────────┤
│            │      INFRASTRUCTURE LAYER                           │
│            v               v                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │ │
│  │  │ aiops-db    │ │ aiops-cache │ │ aiops-      │           │ │
│  │  │ (YugabyteDB │ │ (DragonflyDB│ │ common      │           │ │
│  │  │  client)    │ │  client)    │ │ (shared     │           │ │
│  │  └─────────────┘ └─────────────┘ │  types,     │           │ │
│  │                                   │  errors,    │           │ │
│  │                                   │  config)    │           │ │
│  │                                   └─────────────┘           │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### Crate Descriptions

| Crate | Layer | Description | Key Dependencies |
|-------|-------|-------------|------------------|
| `aiops-api` | Application | Axum HTTP routes, request/response types, middleware | aiops-incident, aiops-rules, aiops-topology, aiops-common |
| `aiops-ws` | Application | WebSocket server for real-time event streaming | aiops-cache, aiops-common, tokio-tungstenite |
| `aiops-cli` | Application | Admin CLI for debugging and maintenance | aiops-db, aiops-cache, aiops-common |
| `aiops-worker` | Application | Background job processing (RCA, forecasting, cleanup) | aiops-incident, aiops-correlation, aiops-db |
| `aiops-incident` | Service | Incident CRUD, state machine, SLA tracking | aiops-db, aiops-cache, aiops-common |
| `aiops-correlation` | Service | Temporal, topological, and pattern correlation | aiops-topology, aiops-cache, aiops-common |
| `aiops-rules` | Service | Rule storage, evaluation engine, suppression | aiops-db, aiops-cache, aiops-common |
| `aiops-topology` | Service | Service dependency graph, blast radius analysis | aiops-db, aiops-cache, aiops-common |
| `aiops-remediation` | Service | Playbook management, action execution, verification | aiops-incident, aiops-db, aiops-common |
| `aiops-cost` | Service | Cost data aggregation, optimization recommendations | aiops-db, aiops-common |
| `aiops-security` | Service | Vulnerability scanning, finding management, compliance | aiops-db, aiops-common |
| `aiops-forecast` | Service | Forecasting coordination with Python AI brain | aiops-db, aiops-common, reqwest |
| `aiops-db` | Infrastructure | YugabyteDB connection pool, migrations, query builder | sqlx, aiops-common |
| `aiops-cache` | Infrastructure | DragonflyDB client, stream management, pub/sub | redis-rs, aiops-common |
| `aiops-common` | Infrastructure | Shared types, error handling, config, tenant context | serde, thiserror, config-rs |

---

## 2. Python AI Brain Module Design

The Python AI brain is a FastAPI application organized into modules that provide ML inference capabilities.

### Module Architecture

```
ai_brain/
├── main.py                    # FastAPI app initialization, middleware
├── config.py                  # Configuration management
├── api/
│   ├── v1/
│   │   ├── anomaly.py         # POST /detect, POST /batch-detect
│   │   ├── rca.py             # POST /analyze
│   │   ├── forecast.py        # POST /predict
│   │   ├── threshold.py       # GET/PUT /thresholds
│   │   └── health.py          # GET /health
├── models/
│   ├── anomaly/
│   │   ├── zscore.py          # Z-Score detector
│   │   ├── iqr.py             # IQR detector
│   │   ├── moving_avg.py      # Moving Average detector
│   │   ├── isolation_forest.py# Isolation Forest detector
│   │   ├── lstm.py            # LSTM neural network detector
│   │   └── ensemble.py        # Score fusion engine
│   ├── rca/
│   │   ├── evidence.py        # Evidence collection from metrics/logs/traces
│   │   ├── causal.py          # Granger causality, transfer entropy
│   │   ├── llm_analyzer.py    # LLM-powered analysis and summarization
│   │   └── similarity.py      # Historical incident matching
│   ├── forecast/
│   │   ├── prophet_model.py   # Facebook Prophet forecasting
│   │   ├── arima_model.py     # ARIMA/SARIMA forecasting
│   │   ├── neural_model.py    # Neural network forecasting (N-BEATS)
│   │   └── ensemble.py        # Forecast ensemble and selection
│   └── threshold/
│       ├── adaptive.py        # Adaptive threshold engine
│       └── seasonal.py        # Seasonal adjustment
├── data/
│   ├── loaders.py             # Data loading from VictoriaMetrics, Quickwit
│   ├── preprocessors.py       # Feature extraction, normalization
│   └── cache.py               # Model and result caching
├── storage/
│   ├── model_store.py         # RustFS model artifact management
│   └── result_store.py        # YugabyteDB result persistence
└── utils/
    ├── metrics.py             # Prometheus metrics for the AI brain
    ├── logging.py             # Structured logging
    └── tenant.py              # Tenant context management
```

### Key Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| fastapi | 0.109+ | Web framework |
| scikit-learn | 1.4+ | Isolation Forest, preprocessing |
| torch | 2.2+ | LSTM neural networks, N-BEATS |
| prophet | 1.1+ | Time series forecasting |
| statsmodels | 0.14+ | ARIMA/SARIMA, Granger causality |
| numpy | 1.26+ | Numerical computation |
| pandas | 2.2+ | Data manipulation |
| httpx | 0.27+ | Async HTTP client for VictoriaMetrics/Quickwit |

---

## 3. Anomaly Detection Algorithms

### 3.1 Z-Score Detector

The Z-Score detector identifies data points that deviate significantly from the mean in terms of standard deviations.

**Algorithm:**
```
z_score = (x - μ) / σ

Where:
  x = current metric value
  μ = rolling mean (configurable window, default 1 hour)
  σ = rolling standard deviation

Anomaly score = min(|z_score| / z_threshold, 1.0)

Default z_threshold = 3.0 (configurable per metric)
```

**Strengths:** Fast computation, intuitive interpretation, works well for Gaussian-distributed metrics.

**Weaknesses:** Assumes normal distribution, sensitive to outliers in the rolling window.

### 3.2 IQR Detector

The IQR detector uses interquartile range to identify outliers, robust against non-Gaussian distributions.

**Algorithm:**
```
Q1 = 25th percentile of rolling window
Q3 = 75th percentile of rolling window
IQR = Q3 - Q1
lower_bound = Q1 - (k * IQR)
upper_bound = Q3 + (k * IQR)

distance = max(lower_bound - x, x - upper_bound, 0)
anomaly_score = min(distance / (k * IQR), 1.0)

Default k = 1.5 (configurable per metric)
```

**Strengths:** Robust to outliers, works with skewed distributions.

**Weaknesses:** Less sensitive to subtle deviations in normally distributed data.

### 3.3 Moving Average Detector

The Moving Average detector compares current values against a smoothed trend line.

**Algorithm:**
```
ma = exponential_moving_average(values, span=window_size)
deviation = |x - ma| / ma
anomaly_score = min(deviation / deviation_threshold, 1.0)

Default window_size = 60 data points
Default deviation_threshold = 0.3 (30% deviation)
```

**Strengths:** Excellent for trend detection, smooth and stable.

**Weaknesses:** Lags behind rapid changes, window size selection is critical.

### 3.4 Isolation Forest Detector

Isolation Forest isolates anomalies by randomly partitioning feature space. Anomalies require fewer partitions to isolate.

**Algorithm:**
```
Features per data point:
  - Current value
  - Rate of change (1m, 5m, 15m)
  - Rolling mean (5m, 15m, 1h)
  - Rolling stddev (5m, 15m, 1h)
  - Hour of day (cyclical encoding)
  - Day of week (cyclical encoding)

Model: sklearn.ensemble.IsolationForest(
    n_estimators=100,
    contamination=0.01,
    max_features=0.8
)

anomaly_score = -1 * model.decision_function(features)
# Normalized to [0, 1] via min-max scaling
```

**Strengths:** Multi-dimensional, no distribution assumptions, handles complex patterns.

**Weaknesses:** Requires periodic retraining, higher latency than statistical methods.

### 3.5 LSTM Neural Network Detector

The LSTM detector learns temporal patterns and flags deviations from the predicted sequence.

**Architecture:**
```
Input: Sequence of 60 timesteps, each with 8 features
  → LSTM(128 units, return_sequences=True)
  → Dropout(0.2)
  → LSTM(64 units)
  → Dropout(0.2)
  → Dense(32, activation='relu')
  → Dense(1, activation='linear')  # Predicted next value

reconstruction_error = |predicted - actual|
anomaly_score = min(reconstruction_error / error_threshold, 1.0)
```

**Strengths:** Captures long-term temporal dependencies, seasonal patterns.

**Weaknesses:** Highest latency, requires GPU for training, needs significant historical data.

---

## 4. Event Correlation Algorithm

The correlation engine combines three strategies to group related events.

### Temporal Correlation

```python
def temporal_correlate(events, window_ms=300000):
    """Group events occurring within the time window."""
    sorted_events = sort_by_timestamp(events)
    groups = []
    current_group = [sorted_events[0]]

    for event in sorted_events[1:]:
        if event.timestamp - current_group[-1].timestamp <= window_ms:
            current_group.append(event)
        else:
            groups.append(current_group)
            current_group = [event]

    groups.append(current_group)
    return groups
```

### Topological Correlation

```python
def topological_correlate(events, topology_graph):
    """Group events on topologically connected services."""
    groups = []
    visited = set()

    for event in events:
        if event.service_id in visited:
            continue
        connected = topology_graph.get_connected_component(event.service_id)
        group = [e for e in events if e.service_id in connected]
        for e in group:
            visited.add(e.service_id)
        groups.append(group)

    return groups
```

### Combined Scoring

```
correlation_score = (
    w_temporal * temporal_similarity +
    w_topological * topological_proximity +
    w_pattern * pattern_match_confidence
)

Where:
  w_temporal = 0.4
  w_topological = 0.35
  w_pattern = 0.25

Events are grouped if correlation_score >= 0.7
```

---

## 5. Adaptive Threshold Calculation

The adaptive threshold engine adjusts detection sensitivity based on historical behavior.

```python
def calculate_adaptive_threshold(metric_id, tenant_id):
    # Fetch historical anomaly scores (last 7 days)
    history = fetch_anomaly_history(metric_id, tenant_id, days=7)

    # Calculate base threshold from score distribution
    base_threshold = np.percentile(history.scores, 95)

    # Apply seasonal adjustment
    hour_of_day = datetime.now().hour
    seasonal_factor = seasonal_model.predict(hour_of_day)
    adjusted = base_threshold * seasonal_factor

    # Apply volatility dampening
    recent_volatility = np.std(history.scores[-60:])
    baseline_volatility = np.std(history.scores)
    volatility_ratio = recent_volatility / max(baseline_volatility, 0.01)
    dampened = adjusted * min(volatility_ratio, 2.0)

    # Clamp to valid range
    return max(0.3, min(dampened, 0.95))
```

---

## 6. Incident State Machine

The incident state machine enforces valid transitions and triggers side effects.

```rust
enum IncidentState {
    Detected,
    Acknowledged,
    Investigating,
    Mitigating,
    Resolved,
    Closed,
}

// Valid transitions
const TRANSITIONS: &[(IncidentState, IncidentState)] = &[
    (Detected, Acknowledged),
    (Acknowledged, Investigating),
    (Investigating, Mitigating),
    (Mitigating, Resolved),
    (Mitigating, Investigating),  // remediation failed
    (Resolved, Closed),
    (Resolved, Investigating),    // issue recurred
];

// Side effects per transition
fn on_transition(from: IncidentState, to: IncidentState, incident: &mut Incident) {
    match (from, to) {
        (Detected, Acknowledged) => {
            incident.acknowledged_at = Some(Utc::now());
            stop_sla_timer(SlaType::Acknowledge, incident);
        }
        (Acknowledged, Investigating) => {
            start_sla_timer(SlaType::Resolve, incident);
            if incident.severity <= Severity::P2 {
                trigger_rca(incident);
            }
        }
        (Mitigating, Resolved) => {
            incident.resolved_at = Some(Utc::now());
            stop_sla_timer(SlaType::Resolve, incident);
            schedule_auto_close(incident, Duration::hours(24));
        }
        (Resolved, Closed) => {
            incident.closed_at = Some(Utc::now());
            generate_postmortem_template(incident);
            finalize_metrics(incident);
        }
        _ => {}
    }
}
```

---

## 7. Remediation Execution Engine

The remediation engine coordinates playbook execution with safety guardrails.

```rust
struct RemediationExecutor {
    action_registry: HashMap<ActionType, Box<dyn ActionHandler>>,
    approval_service: ApprovalService,
    audit_log: AuditLogger,
}

impl RemediationExecutor {
    async fn execute_playbook(&self, playbook: &Playbook, trigger: &TriggerEvent)
        -> Result<ExecutionResult>
    {
        // 1. Check tenant quota (max concurrent remediations)
        self.check_quota(trigger.tenant_id).await?;

        // 2. Check approval policy
        let approval = self.approval_service
            .check_policy(&playbook.approval_policy, trigger)
            .await?;

        match approval {
            Approval::AutoApproved => { /* proceed */ }
            Approval::PendingManual(request_id) => {
                return Ok(ExecutionResult::PendingApproval(request_id));
            }
            Approval::Rejected(reason) => {
                return Ok(ExecutionResult::Rejected(reason));
            }
        }

        // 3. Execute steps sequentially
        for step in &playbook.steps {
            let handler = self.action_registry
                .get(&step.action_type)
                .ok_or(Error::UnknownAction)?;

            let result = handler
                .execute(&step.parameters, trigger.tenant_id)
                .await;

            self.audit_log.log_action(trigger, step, &result).await;

            match result {
                Ok(_) => continue,
                Err(e) if step.retry_count > 0 => {
                    // Retry with exponential backoff
                    for attempt in 1..=step.retry_count {
                        tokio::time::sleep(Duration::from_secs(2u64.pow(attempt))).await;
                        if handler.execute(&step.parameters, trigger.tenant_id).await.is_ok() {
                            break;
                        }
                    }
                }
                Err(e) => return Ok(ExecutionResult::Failed(e.to_string())),
            }
        }

        // 4. Run verification if defined
        if let Some(verify) = &playbook.verification {
            let verified = self.verify_outcome(verify, trigger).await?;
            if !verified {
                return Ok(ExecutionResult::PartialSuccess("Verification failed".into()));
            }
        }

        Ok(ExecutionResult::Success)
    }
}
```

---

## 8. Cost Analysis Data Model

### Database Schema

```sql
-- Cost data aggregated from resource metrics
CREATE TABLE cost_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id TEXT NOT NULL,
    service_name TEXT NOT NULL,
    resource_type TEXT NOT NULL,  -- 'cpu', 'memory', 'disk', 'network'
    period_start TIMESTAMPTZ NOT NULL,
    period_end TIMESTAMPTZ NOT NULL,
    allocated_units DECIMAL NOT NULL,
    used_units_avg DECIMAL NOT NULL,
    used_units_p95 DECIMAL NOT NULL,
    unit_cost DECIMAL NOT NULL,
    total_cost DECIMAL NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Optimization recommendations
CREATE TABLE cost_recommendations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id TEXT NOT NULL,
    service_name TEXT NOT NULL,
    category TEXT NOT NULL,      -- 'right_sizing', 'idle', 'reserved', 'storage_tier'
    current_cost DECIMAL NOT NULL,
    projected_savings DECIMAL NOT NULL,
    confidence DECIMAL NOT NULL, -- 0.0 to 1.0
    risk_level TEXT NOT NULL,    -- 'low', 'medium', 'high'
    details JSONB NOT NULL,
    status TEXT DEFAULT 'pending', -- 'pending', 'applied', 'rejected', 'expired'
    applied_at TIMESTAMPTZ,
    actual_savings DECIMAL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_cost_data_tenant_service ON cost_data(tenant_id, service_name, period_start);
CREATE INDEX idx_cost_rec_tenant_status ON cost_recommendations(tenant_id, status);
```
