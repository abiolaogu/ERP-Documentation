# User Manual: Developer -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Development Environment Setup

### 1.1 Prerequisites

- Go 1.22+ (gateway and target microservices)
- Node.js 18+ (source monolith)
- Docker 24+ and Docker Compose
- PostgreSQL client (psql)
- Git

### 1.2 Clone and Start

```bash
git clone <repository-url> ERP-Church-Management
cd ERP-Church-Management

# Start all services with Docker Compose
docker compose up -d

# Verify health
curl http://localhost:8093/healthz
# Expected: {"module":"ERP-Church-Management","status":"healthy"}
```

### 1.3 Service Ports

| Service | Internal Port | External Port |
|---|---|---|
| Gateway | 8090 | 8093 |
| PostgreSQL | 5432 | 5435 |
| Redis | 6379 | 6381 |
| Redpanda | 9092 | 19094 |
| Each microservice | 8080 | (internal only) |

---

## 2. Repository Structure

```
ERP-Church-Management/
  gateway/
    main.go              # Go API gateway
    main_test.go         # Gateway tests
  services/
    member-service/      # Member domain service
    visitor-service/     # Visitor domain service
    followup-service/    # Follow-up domain service
    giving-service/      # Giving domain service
    event-service/       # Event domain service
    group-service/       # Group domain service
    discipleship-service/# Discipleship domain service
    welfare-service/     # Welfare domain service
    communication-service/# Communication domain service
    kpi-service/         # KPI domain service
    volunteer-service/   # Volunteer domain service
    facility-service/    # Facility domain service
  database/
    migrations/          # SQL migration files
  frontend/
    web/                 # React/Next.js web application
  mobile/                # Flutter mobile application
  source-monolith/       # Original Node.js application
  configs/               # Configuration files
  docker-compose.yml     # Full stack orchestration
  gateway.Dockerfile     # Gateway container build
  Makefile               # Build and run commands
```

---

## 3. Gateway Development

### 3.1 Architecture

The gateway is a single Go file (`gateway/main.go`) using only the standard library. It implements:

```mermaid
flowchart TD
    MAIN["main()"] --> CAP["loadCapabilities()"]
    CAP --> MUX["buildMux()"]
    MUX --> MW1["withCorrelationID()"]
    MW1 --> MW2["requireTenant()"]
    MW2 --> MW3["requireJWT()"]
    MW3 --> MW4["requireEntitlement()"]
    MW4 --> LISTEN["http.ListenAndServe(:8090)"]
```

### 3.2 Adding a New Service

To add a new service to the gateway:

1. Add upstream entry in `serviceRegistry()`:
```go
"newservice": env("CHURCH_UPSTREAM_NEWSERVICE", "http://newservice:8080"),
```

2. The gateway automatically routes `/v1/newservice/...` to the upstream.

3. Add the service to `docker-compose.yml`:
```yaml
newservice:
  build:
    context: ./services/newservice
  environment:
    - PORT=8080
    - DATABASE_URL=postgres://erp:erp@postgres:5432/erp_church_management
    - REDIS_ADDR=redis:6379
    - KAFKA_BROKERS=redpanda:9092
  depends_on:
    - postgres
    - redis
    - redpanda
```

### 3.3 Running Gateway Tests

```bash
cd gateway
go test -v ./...
```

---

## 4. Microservice Development

### 4.1 Service Template

Each Go microservice follows this structure:

```
services/{name}/
  cmd/
    main.go              # Entry point, DI wiring
  internal/
    domain/
      model.go           # Domain entities
      events.go          # Domain events
    handler/
      rest.go            # HTTP handlers
      consumer.go        # Kafka consumers
    repository/
      postgres.go        # Database operations
    service/
      service.go         # Business logic
  Dockerfile
  go.mod
  go.sum
```

### 4.2 Implementing a Handler

```go
func (h *MemberHandler) CreateMember(w http.ResponseWriter, r *http.Request) {
    tenantID := r.Header.Get("X-Tenant-ID")

    var req CreateMemberRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeJSON(w, http.StatusBadRequest, ErrorResponse{
            Success: false,
            Message: "Invalid request body",
        })
        return
    }

    member, err := h.service.Create(r.Context(), tenantID, req)
    if err != nil {
        writeJSON(w, http.StatusInternalServerError, ErrorResponse{
            Success: false,
            Message: "Failed to create member",
            Error:   err.Error(),
        })
        return
    }

    writeJSON(w, http.StatusCreated, SuccessResponse{
        Success: true,
        Message: "Member created successfully",
        Data:    member,
    })
}
```

### 4.3 Database Access Pattern

```go
func (r *MemberRepository) FindByID(ctx context.Context, tenantID, memberID string) (*Member, error) {
    query := `
        SELECT id, tenant_id, membership_id, first_name, last_name, email, phone,
               member_type, member_status, natural_group, created_at, updated_at
        FROM members
        WHERE tenant_id = $1 AND id = $2
    `
    var m Member
    err := r.db.QueryRowContext(ctx, query, tenantID, memberID).Scan(
        &m.ID, &m.TenantID, &m.MembershipID, &m.FirstName, &m.LastName,
        &m.Email, &m.Phone, &m.MemberType, &m.MemberStatus,
        &m.NaturalGroup, &m.CreatedAt, &m.UpdatedAt,
    )
    if err == sql.ErrNoRows {
        return nil, ErrMemberNotFound
    }
    return &m, err
}
```

---

## 5. Source Monolith Development

### 5.1 Running the Monolith

```bash
cd source-monolith
cp .env.example .env
# Edit .env with your database credentials
npm install
npm run migrate
npm run dev
# Server starts on http://localhost:5000
```

### 5.2 Monolith Structure

| Directory | Purpose |
|---|---|
| `controllers/` | Request handlers (27 controllers) |
| `routes/` | Express route definitions (27 route files) |
| `database/models/` | Sequelize models (79 models) |
| `database/migrations/` | Database migrations |
| `services/messaging/` | Communication channel adapters |
| `jobs/` | Cron job definitions |
| `middleware/` | Express middleware |
| `validators/` | Request validators |
| `utils/` | Shared utilities |

### 5.3 Adding a Monolith Route

1. Create model in `database/models/`
2. Create controller in `controllers/`
3. Create route in `routes/`
4. Register route in `server.js`
5. Create validator in `validators/`

---

## 6. Database Migrations

### 6.1 Creating a Migration

```bash
# For target microservices (SQL files)
touch database/migrations/NNNN_description.sql

# For source monolith (Sequelize)
cd source-monolith
npx sequelize-cli migration:generate --name add-new-table
```

### 6.2 Running Migrations

```bash
# Source monolith
cd source-monolith && npm run migrate

# Direct SQL
psql -h localhost -p 5435 -U erp -d erp_church_management -f database/migrations/0001_initial_core.sql
```

---

## 7. Kafka/Redpanda Development

### 7.1 Producing Events

```go
func (p *EventPublisher) PublishVisitorCreated(ctx context.Context, visitor *Visitor) error {
    event := DomainEvent{
        EventID:       uuid.New().String(),
        EventType:     "visitor.created",
        TenantID:      visitor.TenantID,
        Timestamp:     time.Now().UTC(),
        CorrelationID: ctx.Value("correlation_id").(string),
        Source:        "visitor-service",
        Payload:       visitor,
    }
    payload, _ := json.Marshal(event)
    return p.writer.WriteMessages(ctx, kafka.Message{
        Topic: "church.visitor.events",
        Key:   []byte(visitor.TenantID),
        Value: payload,
    })
}
```

### 7.2 Consuming Events

```go
func (c *FollowupConsumer) Start(ctx context.Context) {
    reader := kafka.NewReader(kafka.ReaderConfig{
        Brokers:  c.brokers,
        Topic:    "church.visitor.events",
        GroupID:  "followup-service",
    })
    for {
        msg, err := reader.ReadMessage(ctx)
        if err != nil { break }
        var event DomainEvent
        json.Unmarshal(msg.Value, &event)
        switch event.EventType {
        case "visitor.created":
            c.handleVisitorCreated(ctx, event)
        }
    }
}
```

---

## 8. Testing

### 8.1 Go Tests

```bash
# Gateway tests
cd gateway && go test -v ./...

# Service tests
cd services/member-service && go test -v ./...
```

### 8.2 Node.js Tests (Monolith)

```bash
cd source-monolith
npm test                    # Run all tests
npm test -- --coverage      # With coverage report
```

### 8.3 API Integration Tests

```bash
# Start full stack
docker compose up -d

# Run integration tests
cd tests && go test -v -tags=integration ./...
```

---

## 9. Debugging

### 9.1 Viewing Service Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f member-service

# Gateway only
docker compose logs -f gateway
```

### 9.2 Database Access

```bash
psql -h localhost -p 5435 -U erp -d erp_church_management
```

### 9.3 Redis Access

```bash
redis-cli -h localhost -p 6381
```

### 9.4 Redpanda Topic Inspection

```bash
docker compose exec redpanda rpk topic list
docker compose exec redpanda rpk topic consume church.visitor.events --num 10
```

---

## 10. Coding Standards

| Area | Standard |
|---|---|
| Go formatting | `gofmt` / `goimports` |
| Go linting | `golangci-lint` |
| Node.js | ESLint with Airbnb config |
| API naming | Kebab-case for URLs, camelCase for JSON |
| Database naming | Snake_case for tables and columns |
| Commit messages | Conventional Commits format |
| Branch naming | `feature/CHM-123-description`, `fix/CHM-456-description` |
