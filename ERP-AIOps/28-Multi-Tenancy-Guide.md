# ERP-AIOps Multi-Tenancy Guide

> **Document ID:** ERP-AIOPS-MT-028
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [31-SECURITY.md](./31-SECURITY.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## 1. Multi-Tenancy Architecture Overview

ERP-AIOps is designed as a multi-tenant platform where each tenant (organization) has complete data isolation while sharing the underlying infrastructure. Tenant isolation is enforced at every layer of the stack.

```
┌──────────────────────────────────────────────────────────────────┐
│                     REQUEST FLOW                                 │
│                                                                  │
│  Client Request                                                  │
│       │                                                          │
│       v                                                          │
│  ┌─────────────────────┐                                        │
│  │  Go Gateway (:8090) │                                        │
│  │  1. Validate JWT    │                                        │
│  │  2. Extract tenant  │                                        │
│  │     from claims     │                                        │
│  │  3. Set X-Tenant-ID │                                        │
│  │  4. Apply rate limit│                                        │
│  │     (per tenant)    │                                        │
│  └──────────┬──────────┘                                        │
│             │  X-Tenant-ID: tenant-001                          │
│             v                                                    │
│  ┌─────────────────────┐                                        │
│  │  Rust API (:8080)   │                                        │
│  │  1. Read X-Tenant-ID│                                        │
│  │  2. Set tenant ctx  │                                        │
│  │  3. Filter all DB   │                                        │
│  │     queries by      │                                        │
│  │     tenant_id       │                                        │
│  │  4. Scope cache keys│                                        │
│  └──────────┬──────────┘                                        │
│             │                                                    │
│       ┌─────┴──────┐                                            │
│       v            v                                            │
│  ┌──────────┐  ┌──────────┐                                    │
│  │YugabyteDB│  │DragonflyDB│                                    │
│  │ WHERE    │  │ Key prefix│                                    │
│  │ tenant_id│  │ tenant:   │                                    │
│  │ = ?      │  │ {id}:...  │                                    │
│  └──────────┘  └──────────┘                                    │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Tenant Isolation in Rust API (X-Tenant-ID Header)

### Tenant Context Extraction

The Go gateway extracts the tenant ID from the JWT claims and injects it as the `X-Tenant-ID` header. The Rust API reads this header and propagates it through the request context.

```rust
// Middleware: Extract tenant from X-Tenant-ID header
pub async fn tenant_middleware(
    mut req: Request<Body>,
    next: Next,
) -> Result<Response, StatusCode> {
    let tenant_id = req
        .headers()
        .get("X-Tenant-ID")
        .and_then(|v| v.to_str().ok())
        .map(String::from)
        .ok_or(StatusCode::FORBIDDEN)?;

    // Validate tenant exists and is active
    let tenant = validate_tenant(&tenant_id)
        .await
        .map_err(|_| StatusCode::FORBIDDEN)?;

    // Insert into request extensions for downstream handlers
    req.extensions_mut().insert(TenantContext {
        id: tenant_id,
        tier: tenant.tier,
        config: tenant.config,
    });

    Ok(next.run(req).await)
}
```

### Automatic Query Scoping

All database queries are automatically scoped to the tenant. The `TenantContext` is used by the query builder to append `WHERE tenant_id = $1` to every query.

```rust
// Repository layer: All queries automatically scoped
impl IncidentRepository {
    pub async fn list(&self, ctx: &TenantContext, filters: &IncidentFilters)
        -> Result<Vec<Incident>>
    {
        sqlx::query_as!(
            Incident,
            r#"
            SELECT * FROM incidents
            WHERE tenant_id = $1
              AND ($2::text IS NULL OR severity = $2)
              AND ($3::text IS NULL OR state = $3)
            ORDER BY created_at DESC
            LIMIT $4 OFFSET $5
            "#,
            ctx.id,          // Always filtered by tenant
            filters.severity,
            filters.state,
            filters.limit,
            filters.offset,
        )
        .fetch_all(&self.pool)
        .await
    }
}
```

### Row-Level Security (Database Enforcement)

In addition to application-level filtering, YugabyteDB row-level security policies provide a defense-in-depth layer.

```sql
-- Enable RLS on all tenant-scoped tables
ALTER TABLE incidents ENABLE ROW LEVEL SECURITY;
ALTER TABLE anomalies ENABLE ROW LEVEL SECURITY;
ALTER TABLE rules ENABLE ROW LEVEL SECURITY;
ALTER TABLE remediation_playbooks ENABLE ROW LEVEL SECURITY;
ALTER TABLE cost_recommendations ENABLE ROW LEVEL SECURITY;
ALTER TABLE security_findings ENABLE ROW LEVEL SECURITY;

-- Create RLS policy
CREATE POLICY tenant_isolation ON incidents
    USING (tenant_id = current_setting('app.tenant_id'));

CREATE POLICY tenant_isolation ON anomalies
    USING (tenant_id = current_setting('app.tenant_id'));

-- Set tenant context before each request
SET app.tenant_id = 'tenant-001';
```

---

## 3. Per-Tenant ML Model Instances

### Model Isolation Strategy

Each tenant gets isolated ML model instances to prevent cross-tenant data leakage and allow per-tenant tuning.

| Model | Isolation Level | Rationale |
|-------|----------------|-----------|
| Z-Score | Per-tenant rolling statistics | Baselines differ per tenant's workload |
| IQR | Per-tenant percentile tracking | Metric distributions vary by tenant |
| Moving Average | Per-tenant window | Different traffic patterns |
| Isolation Forest | Per-tenant trained model | Contamination rates differ |
| LSTM | Shared base + per-tenant fine-tuning | Transfer learning for efficiency |

### Model Storage Structure

```
rustfs://aiops-models/
├── shared/
│   └── lstm_base/v1.1.0/weights.pt       # Shared base model
├── tenant-001/
│   ├── isolation_forest/v1.2.0/model.pkl
│   ├── lstm_finetuned/v1.0.0/weights.pt
│   └── threshold_config.json
├── tenant-002/
│   ├── isolation_forest/v1.1.0/model.pkl
│   ├── lstm_finetuned/v1.0.0/weights.pt
│   └── threshold_config.json
└── tenant-003/
    └── ...
```

### Per-Tenant Model Loading

```python
class TenantModelManager:
    def __init__(self):
        self._models: Dict[str, Dict[str, Any]] = {}  # tenant_id -> {model_name -> model}

    async def get_model(self, tenant_id: str, model_name: str) -> Any:
        if tenant_id not in self._models:
            self._models[tenant_id] = {}

        if model_name not in self._models[tenant_id]:
            # Load tenant-specific model from RustFS
            model = await self._load_from_storage(tenant_id, model_name)
            self._models[tenant_id][model_name] = model

        return self._models[tenant_id][model_name]

    async def retrain_for_tenant(self, tenant_id: str, model_name: str):
        # Fetch only this tenant's data
        data = await self._fetch_tenant_metrics(tenant_id, days=30)

        # Train model on tenant-specific data
        model = train_model(model_name, data)

        # Save to tenant-specific path
        await self._save_to_storage(tenant_id, model_name, model)
        self._models[tenant_id][model_name] = model
```

---

## 4. Per-Tenant Rule Evaluation

### Rule Scoping

Rules are always scoped to the tenant that created them. A rule created by tenant-001 will only evaluate against tenant-001's events.

```rust
pub async fn evaluate_rules(
    ctx: &TenantContext,
    event: &Event,
    rule_engine: &RuleEngine,
) -> Vec<RuleMatch> {
    // Fetch only this tenant's active rules
    let rules = rule_engine
        .get_active_rules(ctx.id.as_str())
        .await;

    let mut matches = Vec::new();
    for rule in &rules {
        if rule.evaluate(event) {
            matches.push(RuleMatch {
                rule_id: rule.id.clone(),
                tenant_id: ctx.id.clone(),
                event_id: event.id.clone(),
                matched_at: Utc::now(),
            });
        }
    }
    matches
}
```

### System Rules vs Tenant Rules

| Rule Type | Scope | Creator | Editable by Tenant |
|-----------|-------|---------|-------------------|
| System Rules | All tenants | Platform admin | No (view only) |
| Tenant Rules | Single tenant | Tenant admin | Yes |
| Template Rules | Cloned to tenant on creation | Platform admin | Yes (after clone) |

---

## 5. Data Isolation in YugabyteDB

### Schema Design

Every tenant-scoped table includes a `tenant_id TEXT NOT NULL` column as part of the composite primary key or with an enforced index.

```sql
CREATE TABLE incidents (
    id UUID DEFAULT gen_random_uuid(),
    tenant_id TEXT NOT NULL,
    title TEXT NOT NULL,
    severity TEXT NOT NULL,
    state TEXT NOT NULL DEFAULT 'detected',
    -- ... other columns
    created_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (tenant_id, id)  -- Tenant ID is part of PK for distribution
);

-- All queries hit the tenant partition first
CREATE INDEX idx_incidents_tenant_state ON incidents(tenant_id, state, created_at DESC);
CREATE INDEX idx_incidents_tenant_severity ON incidents(tenant_id, severity, created_at DESC);
```

### Data Distribution

YugabyteDB distributes data across tablets using the primary key hash. By including `tenant_id` as the first column in the primary key, data for a single tenant is colocated, improving query performance.

### Cross-Tenant Query Prevention

The application layer never allows queries without a tenant filter. This is enforced by:

1. **Middleware**: Every request must have a tenant context.
2. **Repository layer**: All query methods require `TenantContext` as the first parameter.
3. **RLS policies**: Database-level enforcement as a safety net.
4. **CI tests**: Static analysis checks that all SQL queries include `tenant_id` in WHERE clauses.

---

## 6. Cache Isolation in DragonflyDB

### Key Naming Convention

All cache keys are prefixed with the tenant ID to prevent cross-tenant cache access.

```
Key Pattern: {tenant_id}:{resource_type}:{resource_id}

Examples:
  tenant-001:incident:inc-uuid-123
  tenant-001:anomaly:score:svc-crm:cpu_usage
  tenant-001:topology:graph
  tenant-001:rule:eval:cache:rule-uuid
  tenant-001:session:user-uuid
```

### Cache Operations

```rust
pub struct TenantCache {
    client: redis::Client,
}

impl TenantCache {
    fn tenant_key(&self, tenant_id: &str, key: &str) -> String {
        format!("{}:{}", tenant_id, key)
    }

    pub async fn get<T: DeserializeOwned>(
        &self,
        tenant_id: &str,
        key: &str,
    ) -> Result<Option<T>> {
        let full_key = self.tenant_key(tenant_id, key);
        let result: Option<String> = self.client.get(&full_key).await?;
        match result {
            Some(json) => Ok(Some(serde_json::from_str(&json)?)),
            None => Ok(None),
        }
    }

    pub async fn set<T: Serialize>(
        &self,
        tenant_id: &str,
        key: &str,
        value: &T,
        ttl: Duration,
    ) -> Result<()> {
        let full_key = self.tenant_key(tenant_id, key);
        let json = serde_json::to_string(value)?;
        self.client.set_ex(&full_key, &json, ttl.as_secs() as usize).await?;
        Ok(())
    }

    pub async fn flush_tenant(&self, tenant_id: &str) -> Result<u64> {
        // Only flush keys for this specific tenant
        let pattern = format!("{}:*", tenant_id);
        let keys: Vec<String> = self.client.scan_match(&pattern).await?;
        let count = keys.len() as u64;
        if !keys.is_empty() {
            self.client.del::<_, ()>(&keys).await?;
        }
        Ok(count)
    }
}
```

### Stream Isolation

DragonflyDB Streams used for real-time event processing are also tenant-scoped:

```
Stream: events:{tenant_id}
Stream: anomalies:{tenant_id}
Stream: remediation:{tenant_id}
```

WebSocket connections only subscribe to streams matching their authenticated tenant ID.

---

## 7. Quota Enforcement per Tenant Tier

### Tier Definitions

| Quota | Free | Standard | Enterprise |
|-------|------|----------|------------|
| Monitored Services | 5 | 20 | Unlimited |
| Events/sec | 1,000 | 10,000 | 100,000 |
| Detection Rules | 10 | 50 | Unlimited |
| Remediation Playbooks | 3 | 15 | Unlimited |
| Data Retention (days) | 7 | 30 | 90 |
| API Rate Limit (req/min) | 60 | 300 | 3,000 |

### Quota Enforcement Points

```rust
pub struct QuotaEnforcer {
    cache: TenantCache,
}

impl QuotaEnforcer {
    pub async fn check_event_rate(&self, ctx: &TenantContext) -> Result<()> {
        let key = format!("quota:events:{}:{}", ctx.id, current_minute());
        let count: u64 = self.cache.incr(&key).await?;

        // Set TTL on first increment
        if count == 1 {
            self.cache.expire(&key, 60).await?;
        }

        let limit = match ctx.tier {
            Tier::Free => 1_000,
            Tier::Standard => 10_000,
            Tier::Enterprise => 100_000,
        };

        if count > limit {
            return Err(Error::QuotaExceeded {
                resource: "events_per_second".into(),
                limit,
                current: count,
                tenant_id: ctx.id.clone(),
            });
        }

        Ok(())
    }

    pub async fn check_resource_count(
        &self,
        ctx: &TenantContext,
        resource: &str,
        current_count: u64,
    ) -> Result<()> {
        let limit = self.get_limit(ctx.tier, resource);
        if limit > 0 && current_count >= limit {
            return Err(Error::QuotaExceeded {
                resource: resource.into(),
                limit,
                current: current_count,
                tenant_id: ctx.id.clone(),
            });
        }
        Ok(())
    }
}
```

### Rate Limiting at the Gateway

The Go gateway enforces per-tenant API rate limits using a sliding window counter in DragonflyDB:

```go
func rateLimitMiddleware(cache *redis.Client) gin.HandlerFunc {
    return func(c *gin.Context) {
        tenantID := c.GetHeader("X-Tenant-ID")
        tier := c.GetString("tenant_tier")

        limit := getLimitForTier(tier)  // 60, 300, or 3000

        key := fmt.Sprintf("ratelimit:%s:%d", tenantID, time.Now().Unix()/60)
        count, _ := cache.Incr(c, key).Result()
        cache.Expire(c, key, 2*time.Minute)

        c.Header("X-RateLimit-Limit", strconv.FormatInt(limit, 10))
        c.Header("X-RateLimit-Remaining", strconv.FormatInt(max(0, limit-count), 10))

        if count > limit {
            c.AbortWithStatusJSON(429, gin.H{
                "error":   "rate_limit_exceeded",
                "message": fmt.Sprintf("Rate limit of %d req/min exceeded", limit),
                "retry_after": 60 - (time.Now().Unix() % 60),
            })
            return
        }

        c.Next()
    }
}
```

### Quota Monitoring Dashboard

Tenant administrators can view their current usage vs. limits in **Settings > Usage & Quotas**:

| Resource | Current | Limit | Usage % |
|----------|---------|-------|---------|
| Monitored Services | 12 | 20 | 60% |
| Events/sec (peak today) | 4,200 | 10,000 | 42% |
| Detection Rules | 28 | 50 | 56% |
| Remediation Playbooks | 8 | 15 | 53% |
| Storage Used | 12.5 GB | 50 GB | 25% |
| API Requests (today) | 45,000 | 432,000 | 10% |

Alerts are triggered at 80% and 95% of quota limits.
