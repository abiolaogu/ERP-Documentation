# Training Manual: Developer -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Training Overview

This training manual provides a structured onboarding curriculum for developers joining the ERP-Church-Management team. The training consists of 6 modules covering architecture comprehension, local development setup, service development, testing, and deployment.

---

## 2. Training Modules

```mermaid
flowchart TD
    M1["Module 1: Architecture Deep Dive\n(2 hours)"] --> M2["Module 2: Local Environment Setup\n(1 hour)"]
    M2 --> M3["Module 3: Gateway & Routing\n(2 hours)"]
    M3 --> M4["Module 4: Service Development\n(4 hours)"]
    M4 --> M5["Module 5: Event-Driven Patterns\n(2 hours)"]
    M5 --> M6["Module 6: Testing & Deployment\n(2 hours)"]
```

---

## 3. Module 1: Architecture Deep Dive

### Learning Objectives
- Understand the monolith-to-microservices migration
- Map the 12 bounded contexts
- Understand the gateway middleware chain

### 3.1 Exercise: Trace a Request

Trace the lifecycle of a `POST /v1/visitor/visitors` request:

1. Client sends HTTPS request with JWT and X-Tenant-ID
2. Gateway receives on `:8090`
3. `requireEntitlement()` checks ERP-Platform
4. `requireJWT()` validates Bearer token format
5. `requireTenant()` ensures X-Tenant-ID header
6. `withCorrelationID()` assigns correlation ID
7. `buildMux()` routes `/v1/visitor/...` to `http://visitor-service:8080`
8. Reverse proxy forwards to visitor-service
9. visitor-service processes request, returns response
10. Gateway relays response to client

**Task**: Draw a sequence diagram of this flow on a whiteboard.

### 3.2 Exercise: Service Map

Using the docker-compose.yml, create a diagram showing:
- All 12 services and their dependencies
- Which services connect to PostgreSQL, Redis, and Redpanda
- The gateway's routing table (service name -> upstream URL)

---

## 4. Module 2: Local Environment Setup

### 4.1 Exercise: Full Stack Startup

```bash
# Clone the repository
git clone <repo-url> ERP-Church-Management
cd ERP-Church-Management

# Start all services
docker compose up -d

# Verify health
curl http://localhost:8093/healthz

# Check capabilities
curl http://localhost:8093/v1/capabilities

# Connect to database
psql -h localhost -p 5435 -U erp -d erp_church_management

# Check Redis
redis-cli -h localhost -p 6381 PING

# Check Redpanda
docker compose exec redpanda rpk cluster info
```

### 4.2 Exercise: Explore the Monolith

```bash
cd source-monolith
npm install
cp .env.example .env
# Edit .env with: DB_HOST=localhost, DB_PORT=5435, DB_USER=erp, DB_PASSWORD=erp
npm run dev
# Open http://localhost:5000/health
# Open http://localhost:5000/api
```

**Task**: Call 5 different API endpoints using curl or Postman and document the response shapes.

---

## 5. Module 3: Gateway & Routing

### 5.1 Exercise: Read the Gateway Code

Open `gateway/main.go` and answer these questions:
1. How does the gateway determine which upstream to route to?
2. What happens if an unknown service name appears in the URL?
3. How does `requireEntitlement()` handle ERP-Platform being down?
4. What headers are injected into proxied requests?
5. How is the capabilities document loaded?

### 5.2 Exercise: Add a Mock Service

1. Create a new directory `services/test-service/`
2. Write a minimal Go HTTP server that returns `{"hello": "world"}`
3. Add it to `docker-compose.yml`
4. Register it in the gateway's `serviceRegistry()`
5. Test: `curl http://localhost:8093/v1/test/hello`

---

## 6. Module 4: Service Development

### 6.1 Exercise: Implement Member Search

Port the member search functionality from the monolith to the Go microservice:

**Source** (from `controllers/member.controller.js`):
```javascript
async searchMembers(req, res) {
    const { query } = req.query;
    const members = await db.Member.findAll({
        where: {
            [Op.or]: [
                { firstName: { [Op.iLike]: `%${query}%` } },
                { lastName: { [Op.iLike]: `%${query}%` } },
                { membershipId: { [Op.iLike]: `%${query}%` } }
            ]
        },
        limit: 20
    });
    res.json({ success: true, data: members });
}
```

**Target**: Implement the equivalent in Go:
1. Create handler: `GET /members/search?query=John`
2. Create repository method with parameterized query
3. Ensure tenant_id is included in WHERE clause
4. Return JSON in the standard envelope format

### 6.2 Exercise: Implement Visitor 72-Hour Check

Port the 72-hour status check from `visitor.controller.js`:

1. Calculate the 72-hour window from visit_date
2. Query pending visitors (not contacted, within window)
3. Query completed contacts (contacted, within window)
4. Calculate completion rate
5. Return structured response

---

## 7. Module 5: Event-Driven Patterns

### 7.1 Exercise: Produce and Consume Events

1. **Producer**: In visitor-service, after creating a visitor, publish a `visitor.created` event to `church.visitor.events` topic

2. **Consumer**: In followup-service, consume `visitor.created` events and auto-assign an Account Officer

3. **Test**: Create a visitor via the API and verify:
   - Event appears in Redpanda topic
   - Follow-up service consumed the event
   - Account Officer was assigned

```bash
# Monitor events
docker compose exec redpanda rpk topic consume church.visitor.events
```

### 7.2 Exercise: Implement Outbox Pattern

1. Within the visitor creation transaction, insert into an `outbox_events` table
2. Create a background worker that polls `outbox_events` and publishes to Kafka
3. Mark events as published after successful Kafka write
4. Test that events are reliably delivered even if Kafka is temporarily down

---

## 8. Module 6: Testing & Deployment

### 8.1 Exercise: Write Unit Tests

Write tests for the member-service:
1. Test `CreateMember` handler with valid input -> expect 201
2. Test `CreateMember` handler with missing required field -> expect 400
3. Test `GetMemberByID` with non-existent ID -> expect 404
4. Test `SearchMembers` with matching query -> expect results
5. Test `SearchMembers` with non-matching query -> expect empty array

### 8.2 Exercise: Write Integration Tests

```go
func TestMemberServiceIntegration(t *testing.T) {
    // 1. Start test database (testcontainers)
    // 2. Run migrations
    // 3. Create a member via HTTP
    // 4. Verify member exists in database
    // 5. Search for member
    // 6. Delete member
    // 7. Verify deletion
}
```

### 8.3 Exercise: Build and Push Container

```bash
# Build
docker build -t erp-church-member-service:dev -f services/member-service/Dockerfile services/member-service/

# Run standalone
docker run -p 8080:8080 \
  -e DATABASE_URL=postgres://erp:erp@host.docker.internal:5435/erp_church_management \
  erp-church-member-service:dev

# Test
curl http://localhost:8080/healthz
```

---

## 9. Certification Assessment

| # | Task | Expected Output |
|---|---|---|
| 1 | Start full stack locally | All 15 containers running, healthz returns 200 |
| 2 | Trace a request through gateway | Correct sequence diagram with all middleware steps |
| 3 | Implement a new REST endpoint | Working endpoint with tenant isolation and error handling |
| 4 | Publish and consume a domain event | Event visible in Redpanda, consumer processes it |
| 5 | Write unit and integration tests | Tests pass, coverage > 80% |
| 6 | Build and run a container | Container starts, healthz responds |

---

## 10. Additional Resources

| Resource | Location |
|---|---|
| Architecture Document | `/docs/architecture.md` |
| API Documentation | `/docs/API.md` |
| Database Schema | `/docs/DATABASE.md` |
| Source Monolith README | `/source-monolith/README.md` |
| Docker Compose | `/docker-compose.yml` |
| Gateway Source | `/gateway/main.go` |
