# ERP-HCM Local Environment Setup

## Step-by-Step Guide

---

## 1. Prerequisites

### 1.1 Required Software

| Software | Version | Installation |
|----------|---------|-------------|
| Go | 1.24+ | https://go.dev/dl/ |
| Node.js | 20 LTS | https://nodejs.org/ or `nvm install 20` |
| PostgreSQL | 16+ | `brew install postgresql@16` or Docker |
| Redis | 7+ | `brew install redis` or Docker |
| NATS Server | 2.10+ | `brew install nats-server` or Docker |
| Docker | Latest | https://docs.docker.com/get-docker/ |
| Git | Latest | `brew install git` |

### 1.2 Verify Installations

```bash
go version          # go1.24.x or later
node --version       # v20.x.x
psql --version       # psql (PostgreSQL) 16.x
redis-cli --version  # redis-cli 7.x.x
nats-server --version # nats-server: v2.10.x
docker --version     # Docker version 24.x.x
```

---

## 2. Clone the Repository

```bash
git clone https://github.com/your-org/ERP-HCM.git
cd ERP-HCM
```

---

## 3. Option A: Docker Compose Setup (Recommended)

### 3.1 Create docker-compose.yml

```yaml
version: '3.9'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: peopleforce
      POSTGRES_PASSWORD: localdev123
      POSTGRES_DB: peopleforce
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  nats:
    image: nats:2.10-alpine
    command: "--jetstream --store_dir /data"
    ports:
      - "4222:4222"
      - "8222:8222"
    volumes:
      - natsdata:/data

volumes:
  pgdata:
  natsdata:
```

### 3.2 Start Infrastructure

```bash
docker compose up -d
```

### 3.3 Verify Services

```bash
# PostgreSQL
psql -h localhost -U peopleforce -d peopleforce -c "SELECT version();"

# Redis
redis-cli ping   # Should return PONG

# NATS
curl http://localhost:8222/healthz   # Should return ok
```

---

## 4. Option B: Local Installation

### 4.1 PostgreSQL

```bash
# macOS with Homebrew
brew install postgresql@16
brew services start postgresql@16

# Create database and user
createuser -s peopleforce
createdb -O peopleforce peopleforce
```

### 4.2 Redis

```bash
brew install redis
brew services start redis
```

### 4.3 NATS

```bash
brew install nats-server
nats-server --jetstream --store_dir /tmp/nats-data &
```

---

## 5. Database Setup

### 5.1 Run Migrations

```bash
# Navigate to migration directory
cd imports/hrms_core/migrations

# Run migrations in order (using psql or a migration tool)
# Core schema
psql -h localhost -U peopleforce -d peopleforce -f core/000001_initial_schema.up.sql

# Employee schema
psql -h localhost -U peopleforce -d peopleforce -f employee/000001_employee_management.up.sql
psql -h localhost -U peopleforce -d peopleforce -f employee/000002_employee_management.up.sql
psql -h localhost -U peopleforce -d peopleforce -f employee/000003_document_asset_disciplinary.up.sql

# Payroll schema
psql -h localhost -U peopleforce -d peopleforce -f payroll/000001_payroll_engine.up.sql
psql -h localhost -U peopleforce -d peopleforce -f payroll/000001_payroll_core.up.sql
psql -h localhost -U peopleforce -d peopleforce -f payroll/000002_global_payroll.up.sql
psql -h localhost -U peopleforce -d peopleforce -f payroll/000010_nigerian_payroll.up.sql

# Leave schema
psql -h localhost -U peopleforce -d peopleforce -f leave/000001_leave_management.up.sql

# Attendance schema
psql -h localhost -U peopleforce -d peopleforce -f attendance/000001_attendance_init.up.sql

# Auth schema
psql -h localhost -U peopleforce -d peopleforce -f auth/000001_authentication_authorization.up.sql

# Continue for remaining schemas...
```

---

## 6. Backend Setup

### 6.1 Environment Variables

Create a `.env.local` file (never commit this):

```bash
# Server
export PF_SERVER_HOST=0.0.0.0
export PF_SERVER_PORT=8080
export PF_ENVIRONMENT=development

# Database
export PF_DATABASE_HOST=localhost
export PF_DATABASE_PORT=5432
export PF_DATABASE_USER=peopleforce
export PF_DATABASE_PASSWORD=localdev123
export PF_DATABASE_DATABASE=peopleforce
export PF_DATABASE_SSL_MODE=disable
export PF_DATABASE_MAX_OPEN_CONNS=25
export PF_DATABASE_MAX_IDLE_CONNS=10

# Redis
export PF_REDIS_HOST=localhost
export PF_REDIS_PORT=6379

# NATS
export PF_NATS_URL=nats://localhost:4222

# Auth (generate RSA keys for local dev)
# openssl genrsa -out private.pem 2048
# openssl rsa -in private.pem -pubout -out public.pem
export PF_AUTH_RSA_PRIVATE_KEY="$(cat private.pem)"
export PF_AUTH_RSA_PUBLIC_KEY="$(cat public.pem)"
export PF_AUTH_JWT_ISSUER=peopleforce-dev
export PF_AUTH_ACCESS_TOKEN_EXPIRY=1h
export PF_AUTH_REFRESH_TOKEN_EXPIRY=720h

# CORS
export PF_CORS_ALLOWED_ORIGINS=http://localhost:3000

# Logging
export PF_LOG_LEVEL=debug
export PF_LOG_FORMAT=console
```

### 6.2 Generate RSA Keys (for JWT)

```bash
# Generate private key
openssl genrsa -out private.pem 2048

# Extract public key
openssl rsa -in private.pem -pubout -out public.pem
```

### 6.3 Run the Gateway

```bash
source .env.local
go run cmd/server/main.go
```

### 6.4 Run Individual Services

Open separate terminal tabs for each service:

```bash
# Terminal 1: Employee Service
source .env.local
PORT=8081 go run services/employee-service/main.go

# Terminal 2: Payroll Service
PORT=8082 go run services/payroll-service/main.go

# Terminal 3: Leave Service
PORT=8083 go run services/leave-service/main.go

# ... continue for other services
```

### 6.5 Verify Backend

```bash
# Health check
curl http://localhost:8090/healthz
# {"status":"ok","module":"ERP-HCM"}

# Capabilities
curl http://localhost:8090/v1/capabilities
# {"module":"ERP-HCM","capabilities":["core_hr","payroll","recruitment","performance","attendance","shift_scheduling","field_workforce"]}

# Employee service
curl -H "X-Tenant-ID: 00000000-0000-0000-0000-000000000001" http://localhost:8081/v1/employee
```

---

## 7. Frontend Setup

### 7.1 Install Dependencies

```bash
cd imports/hrms_core/web/frontend
npm install
```

### 7.2 Configure Environment

Create `.env.local` in the frontend directory:

```bash
NEXT_PUBLIC_API_URL=http://localhost:8090
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-for-local-dev
```

### 7.3 Run Development Server

```bash
npm run dev
# Open http://localhost:3000
```

### 7.4 Build for Production

```bash
npm run build
npm start
```

---

## 8. Running Tests

### 8.1 Backend Tests

```bash
# Unit tests
cd /path/to/ERP-HCM
make test

# Run specific package tests
go test ./imports/hrms_core/internal/payroll/engine/nigeria/...
go test ./imports/hrms_core/internal/attendance/domain/...

# Integration tests (Docker required)
make test-integration

# With coverage
go test -coverprofile=coverage.out ./imports/hrms_core/internal/...
go tool cover -html=coverage.out
```

### 8.2 Frontend Tests

```bash
cd imports/hrms_core/web/frontend

# Unit tests
npm test

# Watch mode
npm run test:watch

# E2E tests (requires running backend)
npm run test:e2e

# Type checking
npm run type-check

# Linting
npm run lint
```

---

## 9. Troubleshooting

### 9.1 Common Issues

| Issue | Solution |
|-------|---------|
| `connection refused` to PostgreSQL | Ensure PostgreSQL is running: `brew services list` or `docker ps` |
| `PONG` not returned from Redis | Start Redis: `brew services start redis` or check Docker |
| JWT validation fails | Ensure RSA keys are properly loaded in env vars |
| CORS errors in browser | Verify `PF_CORS_ALLOWED_ORIGINS` includes `http://localhost:3000` |
| Migration fails | Check if schema already exists; migrations should be idempotent |
| Port already in use | Use `lsof -i :8080` to find and kill the conflicting process |

### 9.2 Useful Commands

```bash
# Reset database
dropdb peopleforce && createdb -O peopleforce peopleforce

# Clear Redis cache
redis-cli FLUSHDB

# Check NATS streams
nats stream ls

# View Docker logs
docker compose logs -f postgres
```

---

## 10. IDE Setup

### 10.1 VS Code Extensions

- Go (official Go extension)
- ESLint
- Tailwind CSS IntelliSense
- Prettier
- PostgreSQL (for SQL syntax)
- Thunder Client (API testing)

### 10.2 GoLand/IntelliJ

- Enable Go modules support
- Set GOROOT to Go 1.24+ installation
- Configure database connection to local PostgreSQL
