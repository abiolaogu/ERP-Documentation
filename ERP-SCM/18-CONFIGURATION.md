# ERP-SCM Configuration Reference

## 1. Overview

ERP-SCM uses a hierarchical configuration system: default values in code, overridden by environment variables, overridden by configuration files. All configuration is managed through Pydantic Settings for type safety and validation.

---

## 2. Configuration Hierarchy

```mermaid
flowchart TB
    A["Code Defaults<br/>(lowest priority)"] --> B["Config Files<br/>(.env / YAML)"]
    B --> C["Environment Variables<br/>(highest priority)"]
    C --> D["Runtime Settings<br/>(Feature Flags)"]
```

---

## 3. Core Application Settings

```python
class Settings(BaseSettings):
    # Application
    APP_NAME: str = "AI Supply Chain Management"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = True
    LOG_LEVEL: str = "INFO"

    # Database
    DATABASE_URL: str = "sqlite:///./scm.db"
    DB_POOL_SIZE: int = 20
    DB_MAX_OVERFLOW: int = 30
    DB_POOL_TIMEOUT: int = 30
    DB_POOL_RECYCLE: int = 3600
    DB_ECHO_SQL: bool = False

    # Redis
    REDIS_URL: str = "redis://localhost:6379/0"
    REDIS_CACHE_TTL: int = 60

    # Event Bus
    KAFKA_BOOTSTRAP_SERVERS: str = "localhost:9092"
    KAFKA_CONSUMER_GROUP: str = "erp-scm"
    KAFKA_AUTO_OFFSET_RESET: str = "latest"

    # Security
    SECRET_KEY: str = "scm-secret-key-change-in-production"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 1440
    IAM_ISSUER: str = ""
    IAM_JWKS_URI: str = ""

    # CORS
    CORS_ORIGINS: list = ["http://localhost:5173", "http://localhost:3000"]

    # AI/ML Configuration
    AI_MODEL_RETRAIN_HOURS: int = 24
    DEMAND_FORECAST_HORIZON_DAYS: int = 30
    ANOMALY_DETECTION_THRESHOLD: float = 0.05
    ANOMALY_Z_THRESHOLD: float = 2.5
    FORECAST_ES_ALPHA: float = 0.3
    FORECAST_ES_BETA: float = 0.1
    FORECAST_ES_WEIGHT: float = 0.4
    FORECAST_RF_WEIGHT: float = 0.6
    FORECAST_RF_ESTIMATORS: int = 100
    FORECAST_RF_MAX_DEPTH: int = 10
    FORECAST_CONFIDENCE_LEVEL: float = 0.95
    ROUTE_OPTIMIZATION_SPEED_KMH: float = 60.0
    ISOLATION_FOREST_CONTAMINATION: float = 0.1

    # Supplier Risk Weights
    SUPPLIER_RISK_WEIGHTS: dict = {
        "delivery_reliability": 0.30,
        "quality_score": 0.25,
        "price_competitiveness": 0.20,
        "financial_stability": 0.15,
        "communication": 0.10,
    }

    # Inventory Settings
    DEFAULT_REORDER_POINT: int = 10
    DEFAULT_REORDER_QUANTITY: int = 50
    DEFAULT_SAFETY_STOCK_Z: float = 1.65  # 95% service level
    DEFAULT_ORDERING_COST: float = 50.0
    DEFAULT_HOLDING_COST_RATE: float = 0.25
    OVERSTOCK_MULTIPLIER: float = 5.0
    DEAD_STOCK_DAYS: int = 90

    # Warehouse Settings
    DEFAULT_PICK_STRATEGY: str = "wave"
    CYCLE_COUNT_VARIANCE_THRESHOLD: float = 0.05
    PUTAWAY_OPTIMIZATION_ENABLED: bool = True

    # Manufacturing Settings
    MRP_PLANNING_HORIZON_DAYS: int = 90
    MRP_BUCKET_SIZE: str = "weekly"  # daily, weekly, monthly
    DEFAULT_PRODUCTION_TYPE: str = "discrete"

    # Fleet Settings
    GPS_UPDATE_INTERVAL_SECONDS: int = 30
    MAINTENANCE_ALERT_DAYS_BEFORE: int = 14
    FUEL_EFFICIENCY_UNIT: str = "l_per_100km"

    # File Upload
    MAX_UPLOAD_SIZE_MB: int = 10
    ALLOWED_FILE_TYPES: list = ["pdf", "csv", "xlsx", "jpg", "png"]
    STORAGE_BACKEND: str = "local"  # local, s3, minio
    S3_BUCKET: str = "erp-scm-documents"
    S3_ENDPOINT: str = ""

    # Notifications
    SMTP_HOST: str = ""
    SMTP_PORT: int = 587
    SMTP_USER: str = ""
    SMTP_PASSWORD: str = ""
    NOTIFICATION_FROM_EMAIL: str = "scm@company.com"

    class Config:
        env_file = ".env"
        extra = "ignore"
```

---

## 4. Environment-Specific Configuration

### 4.1 Development

```env
# .env.development
DEBUG=true
LOG_LEVEL=DEBUG
DATABASE_URL=sqlite:///./scm.db
CORS_ORIGINS=["http://localhost:5173","http://localhost:3000"]
AI_MODEL_RETRAIN_HOURS=1
DB_ECHO_SQL=true
```

### 4.2 Staging

```env
# .env.staging
DEBUG=false
LOG_LEVEL=INFO
DATABASE_URL=postgresql://scm_app:password@pg-staging:5432/scm
REDIS_URL=redis://redis-staging:6379/0
KAFKA_BOOTSTRAP_SERVERS=redpanda-staging:9092
CORS_ORIGINS=["https://scm-staging.company.com"]
AI_MODEL_RETRAIN_HOURS=12
```

### 4.3 Production

```env
# .env.production
DEBUG=false
LOG_LEVEL=WARNING
DATABASE_URL=postgresql://scm_app:${DB_PASSWORD}@pg-primary:5432/scm
REDIS_URL=redis://redis-sentinel:26379/0
KAFKA_BOOTSTRAP_SERVERS=redpanda-0:9092,redpanda-1:9092,redpanda-2:9092
CORS_ORIGINS=["https://scm.company.com","https://portal.company.com"]
AI_MODEL_RETRAIN_HOURS=24
SECRET_KEY=${JWT_SECRET_FROM_VAULT}
IAM_ISSUER=https://iam.company.com
IAM_JWKS_URI=https://iam.company.com/.well-known/jwks.json
STORAGE_BACKEND=s3
S3_BUCKET=erp-scm-production
```

---

## 5. Feature Flags

| Flag | Default | Description |
|---|---|---|
| `FF_AI_FORECASTING_ENABLED` | true | Enable/disable AI demand forecasting |
| `FF_AI_ANOMALY_DETECTION_ENABLED` | true | Enable/disable anomaly detection |
| `FF_AI_ROUTE_OPTIMIZATION_ENABLED` | true | Enable/disable route optimization |
| `FF_SUPPLIER_PORTAL_ENABLED` | true | Enable/disable supplier portal |
| `FF_BARCODE_SCANNING_ENABLED` | true | Enable/disable barcode scanning features |
| `FF_MRP_AUTO_RUN` | false | Auto-run MRP on schedule |
| `FF_THREE_WAY_MATCH_AUTO` | true | Auto-match within tolerance |
| `FF_GPS_TRACKING_ENABLED` | true | Enable fleet GPS tracking |
| `FF_SPC_CHARTS_ENABLED` | false | Enable SPC charting feature |
| `FF_CONSENSUS_PLANNING_ENABLED` | false | Enable consensus planning workflow |

---

## 6. Service-Specific Configuration

### 6.1 Demand Planning Service

| Setting | Default | Description |
|---|---|---|
| `FORECAST_MIN_HISTORY_DAYS` | 7 | Minimum days of history for non-baseline forecast |
| `FORECAST_RESAMPLING_FREQ` | `D` | Time-series resampling frequency |
| `FORECAST_BASELINE_VALUE` | 5.0 | Default forecast when insufficient data |
| `FORECAST_MAX_HORIZON_DAYS` | 365 | Maximum allowed forecast horizon |
| `MODEL_CACHE_DIR` | `/tmp/models` | Directory for cached model artifacts |

### 6.2 Warehouse Service

| Setting | Default | Description |
|---|---|---|
| `PUTAWAY_ZONE_PREFERENCE` | `velocity` | Putaway optimization (velocity, proximity, zone_type) |
| `PICK_PATH_ALGORITHM` | `s_shape` | Pick path optimization (s_shape, shortest_path, largest_gap) |
| `MAX_PICK_WAVE_SIZE` | 50 | Maximum orders per pick wave |
| `LABEL_PRINTER_DRIVER` | `zpl` | Shipping label format (zpl, epl, pdf) |

### 6.3 Fleet Service

| Setting | Default | Description |
|---|---|---|
| `GPS_PROVIDER` | `generic_mqtt` | GPS data provider (geotab, samsara, generic_mqtt) |
| `DRIVER_BEHAVIOR_SPEED_THRESHOLD` | 120 | Speed threshold for alert (km/h) |
| `DRIVER_BEHAVIOR_HARSH_BRAKE_G` | 0.5 | G-force threshold for harsh braking |
| `MAINTENANCE_MILEAGE_INTERVAL` | 15000 | Km between scheduled maintenance |
| `MAINTENANCE_TIME_INTERVAL_DAYS` | 90 | Days between scheduled maintenance |

---

## 7. Tenant-Level Configuration

Tenant-specific settings override global defaults:

```json
{
  "tenant_id": "tenant-uuid",
  "settings": {
    "currency": "USD",
    "timezone": "America/New_York",
    "date_format": "MM/DD/YYYY",
    "number_format": "1,234.56",
    "default_warehouse_id": "wh-uuid",
    "procurement_approval_thresholds": [5000, 25000, 100000],
    "three_way_match_tolerance_pct": 2.0,
    "forecast_auto_generate": true,
    "anomaly_notification_emails": ["ops@company.com"],
    "supplier_scorecard_frequency": "monthly"
  }
}
```
